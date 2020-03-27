---
title: Kubernetes Pods
date: 2020-03-20 17:40:23
tags:
  - pods
categories:
  - Kubernetes
---

### 容器内获取pod信息
- 环境变量，将pod信息注入为环境变量
```bash
env：
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: POD_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
```
- Volume挂载方式,类似于环境变量，只是将数据保存为文件并挂载到内部


### Pod生命周期和重启策略
#### 生命周期
- Pending 已创建pod，但还其中容器一个容器镜像还没有创建
- Running Pod内所有容器都创建，且至少有一个容器处于运行状态
- Successed Pod中所有容器均成功执行并退出，且不会再重启
- Failed Pod内所有容器都已退出，单至少有一个容器退出为失败状态
- Unknown 无法获取该Pod的状态

#### 重启策略
- Always 默认，当容器失效时，kubelet自动重启该容器
- OnFailure 容器终止运行且退出码不为0，kubelet自动重启该容器
- Never 无论容器运行状态如何，kubelet都不会重启容器

kubelet重启失效容器的时间间隔以sync-frequency乘以2n，如1，2，4，8倍，最长延时5min，并在成功重启后10min后重置该时间

每种控制器对pod的重启策略要去如下：
- RC和DaemonSet 必须设置为always，需要保证该容器持续运行
- Job OnFailure或者Never，确保容器执行完成后不再重启
- kubelet 在pod失效时自动重启，不论将RestartPolicy设置为什么值，都不会对pod进行健康检查

### 健康检查和服务可用性检查
#### LivenessProbe探针
判断容器是否存活，如果探针探测到容器不健康，则会kill掉容器。如果不包含该探针，则该容器的LivenessProbe探针返回值永远是Success

#### ReadinessProbe探针
判断容器服务是否可用(Ready状态)，达到Ready状态的Pod才可以接收请求

- ExecAction 执行命令
- TCPSocketAction socket的ip和端口检查
- HTTPGetAction 调用http get方法状态码大于等于200且小于400，正常
	```
	- initialDelaySeconds 启动容器后进行首次检查的等待时间，单位为s
	- timeoutSeconds 健康检查发送请求后等待响应的超时时间，单位为s

	```

### 调度策略

#### NodeSelector 
通过node标签和pod的nodeselector属性匹配来定向调到到某个node节点上

#### NodeAffinity亲和性调度
硬亲和requiredDuringSchedulingIgnoredDuringExecution 必须满足指定的规则才可以调度pod到node上
```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity
  namespace: default
  labels:
   app: myapp-nodeselector
   tier: frontend
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:	# 硬亲和性
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - foo
            - bar
```
软亲和preferredDuringSchedulingIgnoredDuringExecution 强调优先满足指定规则，则尝试调到pod到node，但如果没有满足条件，也调度到node，不强制限制
```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity2
  namespace: default
  labels:
   app: myapp-nodeselector
   tier: frontend
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:	# 软亲和性
      - preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - foo
            - bar
        weight: 60
```
**说明**
1. 如果同时定义了nodeSelector和nodeAffinty，那么必须满足两个条件，pod才能运行在指定的node上
2. 如果nodeAffinity指定了多个nodeSelectorTerms，那么其中一个能够匹配成功即可
3. 如果nodeSelectorTerms中有多个matchExpressions，则一个节点必须满足所有的matchExpressions才能运行该Pod

#### PodAffinity亲和性调度
根据在节点上正在运行的pod的标签来判断调度，要求对节点和Pod两个条件进行匹配， 第一个pod启动，其他的pod也在该node启动

requiredDuringSchedulingIgnoredDuringExecution  必须满足指定的规则才可以调度pod到node上
```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-first
  namespace: default
  labels:
   app: myapp-nodeselector
   tier: frontend
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-second
  namespace: default
  labels:
   app: myapp-db
   tier: db
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["sh", "-c", "sleep 3600"]
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - {key: app, operator: In, values: ["myapp-nodeselector"]}
        topologyKey: kubernetes.io/hostname
```
反亲和性
```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-first
  namespace: default
  labels:
   app: myapp-nodeselector
   tier: frontend
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-second
  namespace: default
  labels:
   app: myapp-db
   tier: db
  annotations:
    created-by: "ttxsgoto"
spec:
  containers:
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["sh", "-c", "sleep 3600"]
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - {key: app, operator: In, values: ["myapp-nodeselector"]}
        topologyKey: kubernetes.io/hostname
```
#### Taints and Tolerations 污点和容忍
让Node拒绝Pod的运行，让Pod避开那些不合适的Node，除非pod明确声明能够容忍这些污点，否则无法在这些node上运行

- NoSchedule 仅影响调度过程，对已启动的Pod对象不产生影响，明确声明可以容忍这个taint，否则就不会调度到该node上
- PreferNoSchedule 既影响调度过程，也影响现有的Pod对象，意思为优先，没有对应的node可以运行，也可以调度到node， 可以为NoSchedule的软限制
- NoExecute 即影响调度过程，也影响现在的pod对象，Pod已经在该节点运行，则会被驱逐，如果没有在该节点上运行，则不会再被调度到该节点上

```bash
# 设置Node上 Taint信息
kubectl taint nodes node1 key=value:NoSchedule
 
# pod中声明Toleration
tolerations:
- key: "key"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
或者
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule" 
 
说明：
- 如果不指定operator，默认为Equal
- 空的key配合Exists操作符能够匹配所有的键和值
- 空的effect匹配所有的effect
```

### Pod调度
- RC & Deployment
- DaemonSet
- Job
- Cronjob

#### RC & Deployment
自动部署一个容器的多个副本，以及持续监控副本的数量，在集群内始终维持指定的副本数量
```bash
apiVersion: apps/v1
#apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

#### DaemonSet

- 在每个node上都调度一个pod，适用于监控系统，数据采集等资源的采集
- 新加入的node也同样会自动运行一个pod

```


```

#### Job
批处理调度,仅执行一次的任务，保证处理任务的一个或者多个pod成功结束
```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: k8s-job
spec:
  template:
    spec:
      containers:
      - name: python
        image: python:3.8-alpine
        command: ["python", "-c", "import socket; print(socket.gethostbyname(socket.gethostname()))"]
      restartPolicy: Never
  backoffLimit: 4
```

#### Cronjob
- 在给定时间点只运行一次
- 周期性地给定时间点运行

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: k8s-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: python
            image: python:3.8-alpine
            command: ["python", "-c", "import socket; print(socket.gethostbyname(socket.gethostname()))"]
          restartPolicy: OnFailure
```

### Init Container 初始化容器
应用场景
- 等待其他关联组件正确运行
- 基于环境变量或者配置模板生成配置文件
- 从远程数据库中获取本地所需配置，或者自身注册到某个中央数据库中
- 下载相关依赖包，或者对系统进行一些预配置

init Container和应用容器的区别：
- 先于应用容器执行，如果设置多个init container，按顺序运行，并且当前一个init container运行成功后，在运行下一个，所有都运行成功后，才会运行应用容器
- 在init container的定义中也可以设置资源限制，volume的使用
- 如果多个init container都定义了资源请求，则取最大的值作为所有init container的资源请求值/资源限制值
- 调度算法将基于pod的有效资源请求值/资源限制值进行计算
- init container不能设置readinessProbe探针

```
initContainers:
- name: install
  image: busybox
  command:
  - "sh"
  - "-c"
  - "ls"
```
### Pod升级和回滚
#### Deployment 升级和回滚
- deployment升级镜像

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
或者
kubectl edit deployment/nginx-deployment
  
# 查看升级状态
kubectl rollout status deployment/nginx-deployment
# 查看rs状态
kubectl get rs
```
更新策略说明：
- 在deployment的定义中，可以通过spec.strategy指定pod更新策略，重建(Recreate)和滚动更新(RollingUpdate),默认为RollingUpdate
- spec.strategy.rollingUpdate.maxUnavilable 指定deployment在更新过程中不可用状态的Pod数量的上限
- spec.strategy.rollingUpdate.maxSurge 指定在deployment更新pod的过程中pod总数超过pod期望副本数部分的最大值

```
# 查看deployment部署记录
kubectl rollout history deployment/nginx-deployment
 
# 回滚到上一个部署版本
kubectl rollout undo deployment/nginx-deployment
或者
kubectl rollout undo deployment/nginx-deployment --to-revision=2 # 回滚到之前稳定版本
```
#### 暂停和恢复Deployment部署
复杂的deployment配置修改，可以先暂停deployment的更新操作，然后进行配置修改，在恢复deployment，一次性触发完整的更新操作
```
# 暂停deployment更新操作
kubectl rollout pause deployment/nginx-deployment
# 修改更新操作
# 恢复deployment更新
kubectl rollout resume deploy nginx-deployment
```

### Pod的扩缩容
#### 手动扩缩容机制
```
kubectl scale deployment nginx-deployment --replicas=5
```
### ConfigMap

#### 使用场景
- 通过环境变量获取configmp的内容
- 通过volume挂载的方式将configmap中的内容挂载为容器内部的文件或目录
- 设置容器启动命令的启动参数(引用环境变量)

#### 定义方式
- yaml配置文件
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-demo
data:
  loglevel: info
  datadir: /data/www
```
- 通过命令行方式创建
```
# 直接指定k-v
kubectl create configmap NAME --from-literal=key=value
# 通过目录/文件创建
kubectl create config NAME --from-file=config-dir or config-file
```

#### 使用方式
- 环境变量
```
env:
- name: LOGLEVEL
  valueFrom:
    configMapRef
      name: cm-demo
      key: loglevel
或者
envFrom:
  - configMapRef
    name: cm-demo # 根据cm-demo中的kv自动生成环境变量
    
```
- volumeMount使用ConfigMap
```
volumeMounts:
- name: configpath
  mountPath: /data/config

volumes:
- name: configpath
  configMap:
    # 如果不指定items，则每个item都会生成一个文件名为key的文件
    name: cm-demo
    items:
    - key: serverxml
      path: server.xml
```

#### ConfigMap的限制条件
- configmap必须在pod之前创建
- configmap在相同的namespace中的pod使用
- configmap中的配额管理还未实现
- 通过--manifest-url或者--config创建的静态pod无法使用configmap
- configmap挂载操作，在容器中只能挂载为目录，不能为文件




























