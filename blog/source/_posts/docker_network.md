---
title: Docker 网络模式
date: 2017-02-08 22:05:33
tags:
  - Network
categories:
  - Docker
---
### Docker网络实现原理

Docker 中的网络接口默认都是虚拟的接口。虚拟接口的优势之一是转发效率较高。 Linux 通过在内核中进 行数据复制来实现虚拟接口之间的数据转发，发送接口的发送缓存中的数据包被直接复制到接收接口的接收缓存中。对于本地系统和容器内系统看来就像是一个正常的以太网卡，只是它不需要真正同外部网络设备通信，速度要快很多；Docker 容器网络利用此技术，它在本地主机和容器内分别创建一个虚拟接口，并让它们彼此连通 （这样的一对接口叫做 veth pair ）

Docker 创建一个容器的时候，会执行如下操作：
- 创建一对虚拟接口，分别放到本地主机和新容器中； 
- 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 vethf9； 
- 容器一端放到新容器中，并修改名字作为 eth0，这个接口只在容器的名字空间可见； 
- 从网桥可用地址段中获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 vethf9。 

完成这些之后，容器就可以使用 eth0 虚拟网卡来连接其他容器和其他网络

网络模式需要开启linux系统转发功能，查看linux系统中是否开启转发功能：
```
#sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
# 设置：sysctl -w  net.ipv4.ip_forward=1
```
### 几种网络模式
#### nat
--net=bridge (默认的网桥)，Docker通过宿主机的网桥(docker0)来连通内部和宿主机的网络，实现了容器与宿主机和外界之间的网络通信
![nat](https://ttxsgoto.github.io/img/docker_network/1.png)

Bridge桥接模式的实现步骤主要如下：

（1）Docker Daemon利用veth pair技术，在宿主机上创建两个虚拟网络接口设备，假设为veth0和veth1。而veth pair技术的特性可以保证无论哪一个veth接收到网络报文，都会将报文传输给另一方。

（2）Docker Daemon将veth0附加到Docker Daemon创建的docker0网桥上。保证宿主机的网络报文可以发往veth0。

（3）Docker Daemon将veth1添加到Docker Container所属的namespace下，并被改名为eth0。如此一来，保证宿主机的网络报文若发往veth0，则立即会被eth0接收，实现宿主机到Docker Container网络的联通性；同时，也保证Docker Container单独使用eth0，实现容器网络环境的隔离性。

同时Docker采用NAT（Network Address Translation，网络地址转换）的方式(可自行查询实现原理)，让宿主机以外的世界可以主动将网络报文发送至容器内部。

通过Bridger网桥模式实现：

（1）容器拥有独立、隔离的网络栈

（2）容器和宿主机以外的世界通过NAT建立通信

#### host
--net=host (告诉docker不要将容器网络放在隔离的名字容器中，即不要容器化容器内的网络，该模式下的Docker Container和host宿主机共享同一个网络namespace，即container和宿主机一样，使用宿主机的eth0)
![host](https://ttxsgoto.github.io/img/docker_network/2.png)

Docker Container的host网络模式在实现过程中，由于不需要额外的网桥以及虚拟网卡，故不会涉及docker0以及veth pair。父进程在创建子进程时，如果不使用CLONE_NEWNET这个参数标志，那么创建出的子进程会与父 进程共享同一个网络namespace。Docker就是采用了这个简单的原理，在创建进程启动容器的过程中，没有传入CLONE_NEWNET参数标 志，实现Docker Container与宿主机共享同一个网络环境，即实现host网络模式。

Docker Container的网络模式中，host模式是bridge桥接模式很好的补充。采用host模式的Docker Container，可以直接使用宿主机的IP地址与外界进行通信，若宿主机的eth0是一个公有IP，那么容器也拥有这个公有IP。同时容器内服务的端口也可以使用宿主机的端口，无需额外进行NAT转换。当然，有这样的方便，肯定会损失部分其他的特性，最明显的是Docker Container网络环境隔离性的弱化，即容器不再拥有隔离、独立的网络栈。另外，使用host模式的Docker Container虽然可以让容器内部的服务和传统情况无差别、无改造的使用，但是由于网络隔离性的弱化，该容器会与宿主机共享竞争网络栈的使用；另外，容器内部将不再拥有所有的端口资源，原因是部分端口资源已经被宿主机本身的服务占用，还有部分端口已经用以bridge网络模式容器的端口映射。

#### Other container  
--net=container:NAME_or_ID  (让docker将新建容器的进程放到一个已存在容器的网络栈中，新容器进程有自己的文件系统、进程列表和资源限制，但会和已存在的容器共享IP地址和端口等网络资源，两者进程可以直接通过lo 环回接口通信)
![other](https://ttxsgoto.github.io/img/docker_network/3.png)

上图右侧的Docker Container即采用了other container网络模式，它能使用的网络环境即为左侧Docker Container brdige桥接模式下的网络

Docker Container的other container网络模式在实现过程中，不涉及网桥，同样也不需要创建虚拟网卡veth pair。

完成other container网络模式的创建只需要两个步骤：

(1) 查找other container（即需要被共享网络环境的容器）的网络namespace；

(2) 将新创建的Docker Container（也是需要共享其他网络的容器）的namespace，使用other container的namespace
在这种模式下的Docker Container可以通过localhost来访问namespace下的其他容器，传输效率较高。虽然多个容器共享网络环境，但是多个容器形成的整体依然与宿主机以及其他容器形成网络隔离。另外，这种模式还节约了一定数量的网络资源。但是需要注意的是，它并没有改善容器与宿主机以外世界通信的情况。

#### none     
--net=none   (让Docker将新容器放到隔离的网络栈中，但不进行网络配置，之后用户可以自己进行配置，容器内部只能使用loopback网络设备，不会再有其他的网络资源)

### 网络相关的命令
```
-b BRIDGE or --bridge=BRIDGE --指定容器挂载的网桥
--bip=CIDR --定制 docker0 的掩码
-H SOCKET... or --host=SOCKET... --Docker 服务端接收命令的通道
--icc=true|false --是否支持容器之间进行通信
--ip-forward=true|false --容器之间的通信
--iptables=true|false --禁止 Docker 添加 iptables 规则
--mtu=BYTES --容器网络中的 MTU
```
上述网络模式理论主要来自链接：

http://www.infoq.com/cn/articles/docker-source-code-analysis-part7