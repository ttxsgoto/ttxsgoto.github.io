---
title: Kubernetes Service
date: 2020-03-23 16:30:52
tags:
  - service
categories:
  - Kubernetes
---
### Service功能
- 通过Service为一组具有相同功能的容器应用提供统一的入口地址，并将请求负载分发到后端的各个容器应用上
- Pod的创建和销毁都会实时更新service的Endpoint数据，所以可以动态地对Service的后端Pod进行查询
- 集群外的客户端无法通过pod的ip或者service的虚拟ip和虚拟端口访问，可以通过pod或service的端口号映射到宿主机，将service类型设置为NodePort类型，对service的访问将负载分发到后端的多个pod上

### Service和Pod的关系
- 通过label-selector相关联
- 通过Service实现Pod负载均衡(TCP、UDT 4层负载)

### Service类型
- ClusterIP 集群内使用
- NodePort 对外暴露应用
    ```
     Client -> NodeIP:NodePort -> ClusterIP:ServicePort -> PodIP:containerPort
    ```
- LoadBalancer 用于映射到云服务商提供的LoadBalancer地址

kubernets对应的网络：
- node network
- pod network
- cluster network
- virtual IP

#### ClusterIP
默认，分配一个稳定的IP地址VIP，只能在集群内部访问(同一个namespace内的pod)

#### NodePort
在每个节点上启用一个端口来暴露服务，可以在集群外部访问，也会分配一个稳定内部集群IP地址 NodeIP:NodePort
```python
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:     # 定义pods对应的标签
    app: myapp
  clusterIP: 10.99.99.99
  type: NodePort
  ports:
  - port: 80        # 外部访问端口
    targetPort: 80  # pod内部端口
    nodePort: 30080 # nodeport端口， 对外端口
```
访问 curl http://10.68.2.238:30080/index.html
#### LoadBalancer
和NodePort类似，在每个节点上启用一个端口来暴露服务，另外，kubernetes会请求底层云平台上的负载均衡器，将每个Node(nodeip:nodeport)作为后端添加进去

### Ingress七层代理
- 将不同的URL的访问请求转发到后端不同的Service，以实现HTTP层的业务路由机制
- kubernetes使用一个Ingress策略定义和一个具体的Ingress Controller，两者结合并实现了一个完整的Ingress负载均衡器
- 使用Ingress进行负载分发时，Ingress Controller基于Ingress规则将客户端请求直接转发到Service对应的后端Endpoint(Pod)上，这样会跳过Kube-proxy的转发功能，kube-proxy不再起作用

#### 部署
```
https://github.com/kubernetes/ingress-nginx
 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
# 指定固定访问端口
```
#### Ingress策略配置技巧
Ingress Controller将以Pod的形式运行，监控API Server的ingress接口后端的backend services，如果Service发生变化，则Ingress Controller应自动更新其转发规则
- 转发到单个后端服务上，请求将转发到后端的唯一Service上，这种情况下Ingress无须定义任何rule
- 同一域名下，不同的url路径转发到不同的服务上
    ```
    - mywebsite.com/web 转发到 web-service:80 上
    - mywebset.com/api 转发到api-service:80 上
    ```
- 不同的域名(虚拟主机名)被转发到不同的服务上，这种配置常用于一个网站通过不同的域名或者虚拟主机名提供不同服务的场景，如foo.bar.com域名由service1提供服务，bar.foo.com域名由service2提供服务
- 不使用域名的转发规则 

#### 实例
```python
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingree-myapp
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"    # 说明和controller相对于的是nginx规则
spec:
  rules:
  - host: myapp.ttxsgoto.com    # 
    http:
      paths:
      - path:
        backend:    # 对应后端的地址
          serviceName: myapp    #  指定service的名称相匹配
          servicePort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v2
        ports:
        - name: http
          containerPort: 80
```
### 服务发现
k8s通过服务发现机制能够很容易获得service对应IP和Port
#### 环境变量

当 Pod 运行在 Node 上，kubelet 会为每个活跃的 Service 添加一组环境变量，格式就是{SVCNAME}_SERVICE_HOST和{SVCNAME**_SERVICE_PORT的变量，但注意 Service 的名称需大写，横线被转换成下划\线
 
**缺点**: 依赖的服务必须在 Pod 启动之前就存在，不然是不会出现在环境变量中的

#### DNS
DNS服务监视Kubernetes API,为每个Service创建dns记录用于域名解析
资源记录：
```
kubectl get pods -n kube-system| grep dns  # 查看dns pod
  
kubectl get svc --all-namespaces   # 查看所有名称空间的服务
 
SVC_NAME.NS_NAME.DOMAIN.LTD
svc.cluster.local
redis.default.svc.cluster.local
所有需要使用域名访问的规则SVC_NAME.NS_NAME.svc.cluster.local 
如: myapp.default.svc.cluster.local 使用dns的方式可以访问k8s集群里面任何想要访问的service资源

```
### Headless Service(clusterIP: None)
Headless Service不需要分配一个VIP，而是可以直接以DNS记录的方式解析出被代理Pod的地址, 主要用途 StatefulSet
解析规则：
- namespace: default
- service: foo
- pod: A-0

域名： a-0.foo.default.svc.cluster.local , 通过DNS服务，查找域名foo.default.svc.cluster.local对应的所有SRV记录，获得一个StatefulSet中所有pod名称

每个pod解析到对应的ip，解析名称规格为：
pod_name.service_name.ns_name.cluster.local

不为service设置ClusterIP(入口IP地址)，仅通过label Selector将后端的Pod列表返回给调用的客户端
```python
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:     # 定义pods对应的标签
    app: myapp
  clusterIP: None
  type: NodePort
  ports:
  - port: 80        # 外部访问端口
    targetPort: 80  # pod内部端口
    nodePort: 30080 # nodeport端口， 对外端口
```

