---
title: Kubernetes 核心组件
date: 2020-03-20 16:19:12
tags:
  - k8s
categories:
  - Kubernetes
---

### API Server原理解析
各类资源对象(Pod,RC,Service)的CURD和watch等HTTP REST接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心

- api server 通过一个名为kube-apiserver的进程提供服务，运行在master上，默认情况下，该进程在本机的8080端口或(6443端口)提供rest服务
- 通过kubectl和api server交互
- 提供认证、授权、访问控制、api注册和发现等机制
- 启动代理来访问 kubectl proxy --port=8080
- 每个Node的kubelet每隔一段时间，调用api server的api接口报告自身状态，更新到etcd中
- kube-controller-manager的Node Controller模块和api server交互，实时监控Node信息
- kube-scheduler通过api server的watch接口监听到新建pod副本的信息后，在调度成功后将pod绑定到目标节点上

Proxy API 接口: 在Kubernetes集群外访问某个pod容器的服务(http服务)时，可以用proxy api实现

### Controller Manager

Controller尝试将状态调整为期望的状态， Controller Manager是k8s中各种操作系统的管理者，是集群内部的管理控制中心，是k8s自动化功能的核心
包含8种Controller：
- Replication Controller
- Node Controller
- ResourceQuota Controller
- Namespace Controller
- ServiceAccount Controller
- Token Controller
- Service Controller
- Endpoint Controller

Controller Manager是这些Controller的核心管理者

####  Replication Controller 副本控制器
确保在任何时候集群中某个RC关联的Pod副本数量都保持预设值，需要将pod重启策略设置为always(RestartPolicy=Always),Replication Controller才会管理该pod的操作

功能总结：
- 确保当前集群中有且仅有N个pod实例，N是在RC中定义的Pod副本数量
- 通过调整RC的spec.replicas属性值来实现系统的扩容和缩容
- 通过改变RC中的Pod模板(主要是镜像版本)来实现系统的滚动升级

#### Node Controller
Node Controller通过api server实时获取Node的相关信息，实现管理和监控集群中的各个node的相关控制功能


#### RecourceQuota Controller 资源配额管理 
确保指定的资源对象在任何时候都不会超量占用系统物理资源
- 容器级别，可以对CPU和Memory进行限制
- pod级别，可以对一个pod内所有容器的可用资源进行限制
- Namespace级别，为namespace(多租户)级别的资源限制
	```
	    - pod数量
	    - Replication Controller数量
	    - Service数量
	    - ResourceQuota数量
	    - Secret数量
	    - 可持有的PV数量
	```

#### Namespace Controller
通过api server 可以创建新的Namespace，并将其保存到etcd中，Namespace Controller定时通过api server读取这些Namespace的信息

#### Service Controller && Endpoint Controller
Endpoints表示一个Service对应的所有pod副本的访问地址，Endpoints Controller负责生成和维护所有Endpoints对象的控制器

Service Controller是和外部的接口控制器
- ServiceAccount Controller
- Token Controller

### Scheduler原理
将待调度的Pod(api新创建的Pod、Controller Manager为补足副本而创建的Pod等)按照特定的调度算法和策略绑定(binding)到集群中某个合适的node上，并将绑定信息写入etcd中
- 待调度Pod列表
- 可用Node列表
- 调度算法和策略

目标节点上的kubelet通过API server监听到的scheduler产生的pod绑定事件，然后获取对应的pod清单，下载image和启动容器

默认的调度流程如下：
1. 预选调度过程，即遍历所有目标node，筛选符合要求的候选节点
2. 确定最优节点，采用优选策略计算出每个候选节点的积分，积分最高者胜出

#### 调度算法
- NoDiskConflict     默认
- PodFitsResources    默认
- PodSelectorMatches    默认
- PodFitsHost    默认
- CheckNodeLabelPresence
- CheckServiceAffinity
- PodFitsPorts    默认

#### NoDiskConflict
判断备选的Pod的gcePersistentDisk或AWSElasticBlockStore和备选的节点中已存在的Pod是否存在冲突

#### PodFitsResources
判断备选节点的资源是否满足备选Pod的需求
1. 计算备用pod和节点中已存在的pod容器所需资源(cpu 和内存)的总和
2. 获得备选节点的状态信息，其中包含节点的资源信息
3. 如果备选pod和节点中已存在pod的所有容器的需求资源总和，超出了备选节点拥有的资源，返回false，否则返回true

#### PodSelectorMatches
判断备选节点是否包含备选pod的标签选择器指定的标签
1. 如果pod没有指定spec.nodeSelector标签选择器,则返回true
2. 获取备选节点的标签信息，判断节点是否包含备选pod的标签选择器(spec.nodeSelector)所指定的标签，如果包含返回true，否则返回false

#### PodFitsHost
判断备选pod的spec.nodeName域所指定的节点名称和备选节点的名称是否一致，如果一致返回true，否则返回false

#### CheckNodeLabelPresence
配置文件中指定了该策略，则scheduler会通过RegisterCustomFitPredicate方法注册该策略
1. 读取备选节点的标签列表信息
2. 如果策略配置的标签列表存在于备选节点的标签列表中，且策略配置的presence值为false，则返回false，否则返回true，如果策略配置的标签列表不存在于备选节点的标签列表中，且策略配置的presence值为true，返回false，否则返回true

#### CheckServiceAffinity
该策略用于判断备选节点是否包含策略指定的标签，或包含和备选Pod在相同Service和Namespace下的Pod所在节点的标签列表，如果存在，则返回true，否则返回false

#### PodFitsPorts
判断备选pod所用的端口列表中的端口是否在备选节点中已被占用，如果被占用，则返回false，否则返回true


### Kubelet运行机制解析
每个node上都会启动一个kubelet服务进程，该进程用于处理master下发到本节点的任务，管理pod和pod中的容器，同时通过api定期向master节点汇报节点资源使用情况，并通过cAdvisor监控容器和节点资源

#### 容器健康检查
- LivenessProbe探针 判断容器健康并反馈给kubelet，如果没有则返回值为success
- ReadlinessProbe探针 判断容器是否启动完成，且准备接收请求

#### cAdvisor资源监控
kubelet通过cAdvisor获取其所在节点和容器的数据，cAdvisor自动查找所有在其所在node上的容器，自动采取cpu、内存、文件系统和网络使用的统计信息


### Kube-proxy运行机制解析
在Kubernetes集群中的每个node上都会运行一个kube-proxy服务进程，功能将到某个service的访问请求转发到后端的多个pod实例上

Service的Cluster IP和Nodeport是kube-proxy服务通过IPtables的NAT转换实现，kube-proxy在运行过程中动态创建与Service相关的iptables规则，这些规则实现了将访问服务的请求负载分发到后端pod的功能

核心思想： 通过api server的watch接口实时跟踪service和endpoint的变更信息，并更新对应的iptables规则，client的请求流量则通过iptables的NAT机制"直接路由"到目标pod

IPVS 专门用于高性能负载均衡，并使用更高效的数据结构(hash表)，允许几乎无限的规模扩张，因此被kube-proxy采纳为第三代模式
- 为大型集群提供更好的可扩展性和性能
- 支持比iptables更复杂的复杂均衡算法(最小负载、最少连接、加权等)
- 支持服务器健康检查和连接重试等功能
- 可以动态修改ipset的集合，即使iptables的规则正在使用这个集合

### Etcd
高可用的键值对存储系统，被用作k8s的后端存储，所有集群数据都存储在里面，用于服务发现和集群管理，存了整个集群的状态


### 总结
- API Server  基本的api server 提供和k8s集群的交互aip接口 ， 和etcd的watch交互信息
- Controller Manager  管理控制中心， 对各种资源的管理，主要控制 pod， node， namespace，token等
- Scheduler 调度器， 主要功能 通过预选算法将 pod 调度到node上，然后将工作交给kubelet
- kubelet 每个node上都运行， 主要为pod的管理，并上报每个节点的资源到master
- kube-proxy  server网络， ipvs外部访问的基础，每一个node上
- Etcd 存储数据， 集群状态







