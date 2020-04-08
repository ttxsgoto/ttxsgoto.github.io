---
title: Kubernetes 认证、Serviceaccount && RBAC
date: 2020-04-08 11:47:11
tags:
  - RBAC
  - Serviceaccount
categories:
  - Kubernetes
---
### 用户和用户组
两种连接到API服务器的客户端
- 真实的人(用户)
- Pod（运行在pod中的应用）

pod使用service accounts机制，该机制被创建和存储在集群中作为ServiceAccount资源

### ServiceAccount资源
在单独的命名空间，为每个命名空间自动创建一个默认的ServiceAccount, 都会用一个默认token，存放在secret中


- 每个pod都与一个serviceAccount相关联
- 多个pod可以使用同一个ServiceAccount
- pod只能使用同一个命名空间中的ServiceAccount
- 可通过ServiceAccount提供镜像拉取密钥

```python
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ttxsgoto-read-pods
  namespace: default

或者
kubectl create sa ttxsgoto-read-pods -n default 
```
在pod的manifest文件中，可以用指定账户名的方式将一个serviceAccount赋值给一个pod，如果不显式地指定serviceAccount账号名称，pod会使用在这个命名空间中的默认serviceAccount

```python
# 查看sa
kubectl get sa
  
# 创建sa，会自动创建token认证信息
kubectl create sa foo -o yaml --dry-run
 
# 查看describe 
kubectl describe sa foo
  
# 查看ServiceAccount密钥 （JWT）
kubectl describe secret foo-token-mfl66
 
# 将serviceAccount分配给pod
通过在pod定义文件中的spec.serviceAccountName字段上设置serviceAccount名称来进行分配（需要在pod创建时进行设置，后续不能被修改）
 
# 查看挂载进pod内的token
cat /var/run/secrets/kubernetes.io/serviceaccount/token # 这个token就是对于的serviceAccount对于的token
```

### RBAC
角色定义了可以做什么操作，绑定定义谁可以做这些操作

- Role(角色)和ClusterRole(集群角色),指定了在资源上可以执行哪些动作
- RoleBinding(角色绑定)和ClusterRoleBinding(集群角色绑定)，它将上述角色绑定到特定用户、组或ServiceAccounts上
- 角色绑定的是命名空间的资源
- 集群绑定的是集群级别的资源


#### Role
Role资源定义了哪些操作可以在哪些资源上执行
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: foo
  name: service-reader
rules:
- apiGroups: [""]   # 核心的apiGroup资源
  verbs: ["get", "list"]    # 获取独立的Service并列出所有允许的服务
  resources: ["services"]   # 这条规则和服务有关，必须使用复数的名字
 
# 或者通过命令行
kubectl create role service-reader --verb=get --verb=list --resource=services -n foo
```

#### RoleBinding
RoleBinding将来自不同命名空间中的ServiceAccount绑定到同一个Role中, 可以是user 、group(user/serviceaccount) and serviceaccount
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ttxsgoto-pods-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pods-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: ttxsgoto
 
# 或者通过命令行
kubectl create rolebinding test role-binding --role=pods-reader --user=email@123.com/ --serviceaccount=default:default
 
--user 参数指定用户名
--group 参数绑定角色到组
```

#### Sa-Rolebinding-Role
```python
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ttxsgoto-read-pods
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pods-reader
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ttxsgoto-sa-rolebind-role
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pods-reader
subjects:
- kind: ServiceAccount
  name: ttxsgoto-read-pods
```
#### Sa-Rolebinding-Clusterrole 
绑定clusterRole,这样用户只能访问该namespace对应下的资源
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ttxsgoto-sa-rolebind-cluster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: ttxsgoto-read-pods
  namespace: default
```

#### User-Rolebinding-Clusterrole
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ttxsgoto-reader-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-readers
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: ttxsgoto
 
# 或者通过命令行
kubectl create rolebinding ttxsgoto-reader-pods --clusterrole=cluster-readers --user=ttxsgoto
```

#### Sa-Clusterrolebinding-Clusterrole
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ttxsgoto-sa-clusterbind-cluster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: ttxsgoto-read-pods
  namespace: default
```

#### ClusterRole
ClusterRole是一种集群级资源，它允许访问没有命名空间的资源和非资源型的URL,或者作为单个命名空间内部绑定的公共角色，从而避免必须在每个命名空间中重新定义相同的角色
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-readers
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
 
# 或者通过命令行
kubectl create clusterrole cluster-readers --verb=get,list,watch --resource=pods
```

#### ClusterRoleBinding
```python
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: ttxsgoto-read-all-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-readers
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: ttxsgoto
 
# 或者通过命令行
kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=foo:default
```

### User 
#### User创建
kubernetes中没有User Account的api对象，创建用户需要使用管理员分配的私钥来创建
```python
# 给用户生成私钥
(umask 077; openssl genrsa -out ttxsgoto.key 2048)
# 给私钥生成证书 
openssl req -new -key ttxsgoto.key -out ttxsgoto.csr -subj "/CN=ttxsgoto/O=ttxsgoto" # cn用户名，o组
# 使用ca.crt来签署证书
openssl x509 -req -in ttxsgoto.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out ttxsgoto.crt -days 1000
# 查看证书
openssl x509 -in ttxsgoto.crt -text -noout
 
# 将创建的证书文件和私钥文件在集群中创建新的凭证 set-credentials，将证书添加为认证k8s信息
kubectl config set-credentials ttxsgoto --client-certificate=./ttxsgoto.crt --client-key=./ttxsgoto.key --embed-certs=true
 
# 给用户创建新的上下文 set-context, ttxsgoto@kubernetes 用户名@集群名
kubectl config set-context ttxsgoto@kubernetes --namespace=default --cluster=kubernetes --user=ttxsgoto
 
# use-context 设置当前用户的context
kubectl config use-context ttxsgoto@kubernetes
# 查看use-context使用有权限操作
kubectl get pods --context=ttxsgoto@kubernetes
 
kubectl config view # 查看config信息
kubectl config --help
kubectl config set-cluster/set-context/set-credentials

```
#### User-Rolebinding-Role
```python
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ttxsgoto-user-role
  namespace: default
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"] # 也可以使用['*']
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ttxsgoto-sa-rolebind-role
  namespace: default
subjects:
- kind: User
  name: ttxsgoto # 这里为username 而非context-name
roleRef:
  kind: Role
  name: ttxsgoto-user-role
  apiGroup: rbac.authorization.k8s.io
```
### Role和Binding绑定组合

访问资源 | 使用的角色类型| 使用的绑定类型|
---|---|---
集群级别的资源(node, PV) |ClusterRole| ClusterRoleBinding
非资源型URL(api/healthz) | ClusterRole | ClusterRoleBinding
任何命名空间中的资源(跨所有命名空间的资源) | ClusterRole| ClusterRoleBinding
在具体命名空间中的资源(在多个命名空间中重用这个相同的ClusterRole)| ClusterRole| RoleBinding
在具体命名空间中的资源(Role必须在每个命名空间定义好) | Role| RoleBinding 

### 总结
- API服务器的客户端有真实的用户和pod中运行的应用
- pod中的应用与一个ServiceAccount关联
- 用户和ServiceAccount都与组进行关联
- 默认情况下，pod运行在每个命名空间自动创建的默认ServiceAccount下
- 额外的ServiceAccount可以手动创建，并且和一个pod关联
- ServiceAccount通过配置可以允许只挂载给定pod中受限的Secret列表
- ServiceAccount可以给pod添加镜像拉取密钥
- Role和ClusterRole定义了可以在哪些资源上执行哪些操作
- RoleBinding和ClusterRolebinding将Role和ClusterRole绑定给用户、组合和ServiceAccount
- 每个集群都有默认的ClusterRole和ClusterRoleBinding

