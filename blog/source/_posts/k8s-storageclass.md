---
title: Kubernetes StorageClass
date: 2020-04-08 14:26:29
tags:
  - StorageClass
categories:
  - Kubernetes
---
### StorageClass

创建动态PV，使用时无需先创建好PV

### 创建Provisioner
使用 StorageClass，我们需要安装对应的自动配置程序，比如我们这里存储后端使用的是 nfs，那么我们就需要使用到一个 nfs-client 的自动配置程序，我们也叫Provisioner，这个程序使用我们已经配置好的 nfs 服务器，来自动创建持久卷，也就是自动帮我们创建 PV
- 自动创建的 PV 以${namespace}-${pvcName}-${pvName}格式创建在NFS服务器上
- 而当这个PV被回收后以archieved-${namespace}-${pvcName}-${pvName}这样的命名格式存在 NFS 服务器上

#### 配置Deployment
```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.238.100
            - name: NFS_PATH
              value: /data/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.238.100
            path: /data/k8s
```
#### ServiceAccount
```python
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
 
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```
#### StorageClass
```python
apiVersion: storage.k8s.io/v1
kind: StorageClass  # 声明了一个名为 es-data-db 的StorageClass对象
metadata:
  name: es-data-db
provisioner: fuseim.pri/ifs # 需要和deployment中的PROVISIONER_NAME环境变量值一样
```

### 新建PVC
- 通过annotations属性来标识自动生成PV
```python
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "es-data-db"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```
- 可以设置es-data-db的StorageClass为Kubernetes的默认存储后端
```python
$ kubectl patch storageclass es-data-db -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### 测试
#### Pod
```python
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - "/bin/sh"
    args:
    - "-c"
    - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
    - name: nfs-pvc
      mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: nfs-pvc
    persistentVolumeClaim:
      claimName: test-pvc
```
#### StatefulSet
```python
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nfs-web
spec:
  serviceName: "nginx"
  selector:
    matchLabels:
      app: nfs-web
  replicas: 3
  template:
    metadata:
      labels:
        app: nfs-web
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: es-data-db
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```