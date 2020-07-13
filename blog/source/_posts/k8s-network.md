---
title: Kubernetes Network
date: 2020-07-10 17:48:08
tags:
  - k8s
categories:
  - Kubernetes
---
#### 网络模型
Kubernetes通过一个叫CNI的接口，维护了一个单独的网桥来代替docker0，即CNI网桥， 在宿主机上的设备名称默认是cni0，和docker网络不同的是将docker0网桥替换成CNI网桥

![](https://ttxsgoto.github.io/img/k8s/k8s-flannel-01.jpg)

#### CNI
Kubernetes 创建一个 Pod 的第一步，就是创建并启动一个 Infra 容器，用来“hold”住这个 Pod 的 Network Namespace
CNI 在Kubernetes启动infra容器之后，就直接可以调用CNI网络插件，为这个Infra容器的Network Namespace配置符合预期的网络栈

##### CNI配置流程
/opt/cni/bin存放的CNI插件所需的基本可执行文件

1. 实现网络方案本身
```
- 即flanneld进程的主要逻辑
- 创建和配置flannel.1设备，配置宿主机路由、ARP和FDB表信息
```
2. 实现网络方案对应的CNI插件
```
- 配置Infra容器里的网络栈，并把它连接在CNI网桥上
```

在宿主机上安装flanneld，并在每台宿主机上生成对应的CNI配置文件/etc/cni/net.d/10-flannel.conflist,kubernetes集群会使用这个配置文件来作为容器网络配置方案
```
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,    # 集群里的 Pod 才可以通过它自己的 Service 访问到自己
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```
CRI(Container Runtime Interface，容器运行时接口)实现容器网络相关的逻辑，对应docker项目为dockershim来实现
##### dockershim处理过程
1. dockershim在处理容器网络时，加载/etc/cni/net.d内的文件，并且把plugins字段中的第一个插件作为默认插件
2. 后面的执行过程中，flannel和portmap会按照定义顺序被调用，从而依次完成“配置容器网络”和“配置端口映射”这两步操作

##### CNI插件的工作原理
1. kubelet创建pod时，首先通过dockershim会先调用docker api创建并启动Infra容器，然后执行SetUpPod方法，作用为：为CNI插件准备参数，然后调用CNI插件为Infra容器配置网络
    - 将容器添加到CNI网络中，即通过容器以Veth Pair方式插到CNI网桥上
    -  /run/flannel/subnet.env为宿主机生成的flannel配置文件
    -  /var/lib/cni/flannel 为flannel cni插件把delegate字段以json文件的方式保存下来
2. CNI bridge插件调用CNI ipam插件，为容器分配可用ip地址，同时为容器设置默认路由
3. CNI插件会把容器的ip地址等信息返回给dockershim，然后被kubelet添加到Pod的Status字段中

#### Service的工作原理
Service是由kube-proxy组件，加上iptables共同实现
访问service vip的ip包经过iptables处理后， 就可以变为访问后端Pod的ip包，而这些endpoints对应的iptables规则，正是kube-proxy通过监听pod的变化事件，在宿主机上生成并维护的

**IPVS模式工作原理**
1. 和iptables模式类似，当我们创建service后，kube-proxy会在宿主机上创建一个虚拟网卡(kube-ipvs0)，并为它分配service VIP作为ip地址
2. kube-proxy通过linux的ipvs模块，为这个ip设置对应的ipvs虚拟主机，并设置这些虚拟主机之间的轮询模式(rr)来作为负载均衡策略，可通过ipvsadm -ln查看；IPVS在内核中的实现是具有Netfilter的NAT模式，IPVS不需要在宿主机上为每个pod设置iptables规则，而是把这些规则的处理放在内核态，从而大大降低了维护iptables的这些规则，从而提高了性能
3. IPVS模块只负责负责均衡和代理模块， 其他的包过滤、SNAT等操作，还是依靠iptables来实现，但这些iptables规则数量有限，而且不会随pod数量的增加而增加
4. kube-proxy设置为 -proxy-mode=ipvs开启ipvs功能

