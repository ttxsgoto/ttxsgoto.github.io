---
title: Docker 网络通信
date: 2020-07-09 17:31:20
tags:
  - Network
categories:
  - Docker
---
Docker通信，需要解决的问题有：
- 同一台主机上的不同docker的通信
- 跨主机的不同docker的通信

#### 网桥(bridge)
网桥(Bridge)：在linux中能够起到虚拟交换机的网络设备，工作在数据链路层，主要通过mac地址学习将数据包发送给网桥的不同端口

在docker项目中会默认在宿主机上创建docker0的网桥，只要连接在docker0网桥上的容器，就可以通过它来进行通信

#### Veth Pair
Docker通过一种名为Veth pair的虚拟设备来连接到docker0网桥上

veth pair的特点：被创建出来后，总是以两张虚拟网卡成对出现，从其中一个网卡发出数据包，可以直接从另一个网卡得到数据包，即使这两个网卡不在同一个network namespace里

#### 同一台主机上的不同docker的通信
被限制在network namespace里的容器进程，实际上是通过Veth Pair设备 + 宿主机网桥的方式实现了同其他容器的数据交换

![](https://ttxsgoto.github.io/img/docker_network/docker-network-01.png)

在容器中的eth0为Veth pair设备在容器的一端，主机上vethxxxx为Veth pair的另一端

**容器虚拟网卡和网桥docker0虚拟网络的Veth pair的配对关系查找**
```bash
docker exec -it <container-name> bash -c 'cat /sys/class/net/eth0/iflink'
# 返回 60
grep -l 60 /sys/class/net/veth*/ifindex
# 此时会有如下类似返回
/sys/class/net/veth11d4238/ifindex
# veth11d4238 即主机上的另一半
 
brctl show  	# 查看veth pair都对应在哪个网桥上
ip link show    # 查看veth pair网卡信息
```

#### 跨主机的不同docker的通信
容器都连接在docker0网桥上，而网络插件则在宿主机上创建了一个特殊的设备，docker0和这个设备，通过ip转发(即路由表)进行转发

我们需要在已有的宿主机网络上，再通过软件构建一个覆盖在已有宿主机网络之上的、可以把所有容器连通在一起的虚拟网络。所以，这种技术就被称为：Overlay Network（覆盖网络）

##### Flannel
- VXLAN   虚拟可扩展局域网，内核态实现封装和解封装
- host-gw 主机二层连通，即同一个vlan中
- UDP 	  严重性能问题(用户态和内核态之间的切换)，已弃用 

##### Tunel设备
Flannel为Tunnel设备，为三层的虚拟网络设备，在操作系统内核和用户应用程序之间传递ip包

##### Flannel子网(Subnet)
由flannel管理的容器网络里，一台宿主机上的所有容器，都属于该宿主机被分配的一个“子网”，这些子网与宿主机的对应关系，保存在etcd中

##### Flannel通信 
Flannel将不同宿主机的容器组成一个大的子网，所有容器都在这个大的网络中，即Overlay Network 得以打通

VXLAN覆盖网络的设计： 在现有的三层网络上，“覆盖”一层虚拟的、内核的VXLAN模块负责维护的二层网络，使得连接在这个VXLAN二层网络的主机之间，可以像在同一个局域网(LAN)里自由通信。

VTEP (VXLAN Tunnel End Point 虚拟隧道端点)
flannel.1设备为VXLAN所需的VTEP设备，拥有ip地址也有mac地址
当数据到宿主机时，路由到flannel.1设备进行处理，进入隧道，找到对应的出口(flanneld进程负责维护设备信息)

VTEP设备需之间需要组建一个虚拟的二层网络，通过二层数据帧进行通信
原始ip包+目的MAC地址封装成二层数据帧(通过ARP表得到目的mac地址)

![](https://ttxsgoto.github.io/img/docker_network/docker-network-02.jpg)