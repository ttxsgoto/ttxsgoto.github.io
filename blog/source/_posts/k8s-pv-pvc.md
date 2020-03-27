---
title: Kubernetes 存储PV&PVC
date: 2020-03-23 17:59:02
tags:
  - pv 
  - pvc
categories:
  - Kubernetes
---

### 存储类型
- empryDir 
- hostPath
- PV && PVC

### emptyDir
pod删除，存储卷也删除， 只在节点node本地使用， pod删除存储卷也删除
```python
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    - name: https
      containerPort: 443
    volumeMounts:
    - name: html
      mountPath: /data/web/html/
  volumes:
  - name: html
    emptyDir: {}
```

### hostPath
pod数据存储在宿主机上

```python
apiVersion: V1
kind: Pod
metadata:
  name: pod-vol-hostpath
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernets/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /user/share/nginx/html/
  volumes:
  - name: html
    hostPath:
      path: /data/pod/volume1   # node上的目录
      type: DirectoryOrCreate
```

kubectl explain pods.spec.volumes 

### NFS存储
```python
server:
sudo apt-get install nfs-kernel-server
vim /etc/exports
/data/volume  192.168.238.0/24(rw,no_root_squash)
sudo /etc/init.d/nfs-kernel-server restart
showmount -e    #查看
 
client:
sudo apt-get install nfs-kernel-server
mount -t nfs 192.168.238.100:/data/volume  /data/
 
 
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-nfs
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: html
    nfs:
      path: /data/volume
      server: 192.168.238.100
```

### PV && PVC
#### PV 
PV是对底层网络存储的抽象，将共享存储定义为“资源”，node可以消费这种资源

PV作为存储资源，主要包括存储能力、访问模式、存储类型、回收策略、后端存储类型等关键信息的设置

#### 访问模式
- ReadWriteOnce(RWO): 读写权限，并且只能被当个node挂载
- ReadOnlyMany(ROX): 只读权限，允许被多个Node挂载
- ReadWriteMany(RWX): 读写权限，允许被多个Node挂载

#### 存储类别(Class)
PV可以设定其存储的类别， 通过storageClassName参数指定一个StorageClass资源对象的名称
#### 回收策略(Reclaim Policy)
- Retain, 保留，保留数据，需要手工处理
- 回收空间， 简单清除文件的操作(rm -rf /volume/*)
- 删除， 与PV相连的后端存储完成Volume的删除操作

目前只有NFS和HostPath两种类型的存储支持Recycle策略， AWS EBS/GCE PD/ Azure Disk和Cinder Volumes支持Delete策略

#### PV生命周期的各个阶段
- Available 可用状态， 还没有和某个PVC绑定
- Bound 已和某个PVC绑定
- Released 绑定的PVC已经删除，资源已释放，但没有被集群回收
- Failed 自动资源回收失败

```python
# kubectl get pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  nfs:
    path: /data/volumes
    server: 192.168.238.100
  accessModes: ["ReadWriteMany", "ReadWriteOnce"]
  capacity:
    storage: 10Gi
```
#### PVC 
用户对存储资源的一个“申请”，pvc可以消费pv资源

主要包括存储空间请求、访问模式、pv选择条件和存储类别等信息的设置

- 资源请求(Resources) 描述
- 访问模式(Access Modes) 
- 存储卷模式(Volume Modes)， pvc也可以设置存储卷模式，用于描述希望使用的pv存储卷模式，包括文件系统和块设备
- PV选择条件(Selector), 通过对Label Selector的设置，可使PVC对于系统中已存在的各种PV进行筛选
- 存储类别(Class) PVC在定义时可以设定需要的后端存储的类别(通过storageClassName字段指定)，以减少对后端存储特性的详细信息依赖

```python
# kubectl get pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  namespace: default
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 6Gi
```

注意：
- PVC 和PV都受限于Namespace, pvc在选择pv时受到Namespace的限制
- selector和class都进行设置时， 系统将选择两个条件同时满足的pv来匹配

### PV和PVC的生命周期
- 资源供应
- 资源绑定
- 资源使用
- 资源释放
- 资源回收

### StorageClass 
动态生成PV

用于标记存储资源的特性和性能
StorageClass定义了主要包括名称、后端存储的提供者(provisioner)和后端存储相关的参数配置
