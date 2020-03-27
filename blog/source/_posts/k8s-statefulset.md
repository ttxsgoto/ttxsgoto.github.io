---
title: Kubernetes StatefulSet
date: 2020-03-27 16:28:36
tags:
  - statefulset
categories:
  - Kubernetes
---
#### StatefulSet
用于有状态应用，如(Mysql,redis, Etcd等)，通过某种方式，记录这些状态，然后在pod被重新创建时，能够为新Pod恢复这些状态

#### Headless Service
无头服务 clusterIP: None

Headless Service不需要分配一个VIP，而是可以直接以DNS记录的方式解析出被代理Pod的地址
解析规则：
- namespace: default
- service: foo
- pod: A-0

域名： a-0.foo.default.svc.cluster.local , 通过DNS服务，查找域名foo.default.svc.cluster.local对应的所有SRV记录，获得一个StatefulSet中所有pod名称

每个pod解析到对应的ip，解析名称规格为：
pod_name.service_name.ns_name.cluster.local

#### volumeClaimTemplates
需要绑定PVC，自动创建PVC，但需要事先创建好PV

volumeClaimTemplates 自动完成的功能：
- pod名称空间自动创建pvc
- pod自动定义volume
- 删除pod时 对应的pvc和pv不会被删除

#### 流程说明
- StatefulSet的控制器直接管理pod，而为了区分这些实例，通过pod的名字里事先约定的编号
- 通过Headless Service为这些有编号的pod在dns服务器中生成带有同样编号的DNS记录，只要statefulSet能够保证这些pod名字里面的编号不变，那么service中的dns记录也不会变
- StatefulSet还为每个pod分配并创建了一个同样编号的PVC,这样保证了每个Pod都拥有一个独立的Volume，即使Pod被删除，对应的PV和PVC依然会保留，这样即使重新创建，也可以找到同样编号的PVC，挂载这个PVC对应的Volume，从而获取到以前保存在Volume中的数据


- StatefulSet 这个资源对象针对的就是有状态的应用，比如 MySQL、Redis、Memcached 
- StatefulSet可以控制pod的启动顺序，还可以为每个pod的状态设置唯一标识(在pod名字后面加0，1，2)，这样对于部署、删除、滚动更新等操作都是有序的
- 支持有序滚动更新 kubectl scale sts myapp --replicas=5
- sts.spec.updateStrategy.rollingUpdate.partition : N 表示表示大于等于N的pod会被更新

#### 实例
```python
# kubectl get sts     # 查看statefulset
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
  labels:
    app: myapp-svc
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None   # Headless Service
  selector:
    app: myapp-pod
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec:
  serviceName: myapp-svc    # 使用Serice名称为myapp-svc来保证pod的可解析身份
  replicas: 2
  selector:
    matchLabels:
      app: myapp-pod
  template:
    metadata:
      labels:
        app: myapp-pod
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v1
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: myappdata
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: 
  - metadata:
      name: myappdata
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```
