---
title: Kubernetes 集群部署
date: 2020-03-20 16:42:34
tags:
  - k8s
categories:
  - Kubernetes
---
#### Environment
- 192.168.2.100 (master, ubuntu 16.04)
- 192.168.2.99 (node1, ubuntu 16.04)
- 192.168.2.101(node2, ubuntu 16.04) 

#### Install Docker
```bash
# master, node1, node2
apt-get install apt-transport-https ca-certificates curl software-properties-common -y
 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 
apt-get update -y
apt-get install docker-ce -y
```

#### Install Kubernetes
```bash
# master, node1, node2
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
 
echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list
 
apt-get update -y
apt-get install kubelet kubeadm kubectl -y
```

#### Kubernetes Cluster
```bash
# master
kubeadm init \
  --apiserver-advertise-address=192.168.2.100 \
  --kubernetes-version v1.17.3 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16  --v=6
  
# Install Pod network（CNI）
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
# node1, node2
kubeadm join 192.168.2.100:6443 --token iccmkm.qu60zi1gjdtrqo41 \
    --discovery-token-ca-cert-hash sha256:fffd09fe6d8818acc5fd42ce679661f550a3890e1d97e2f6a6609efa435005ac
  
#查看添加进集群的token命令
kubeadm token create --print-join-command
 
# Install Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
 
默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：
##### begin #####
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30443
  selector:
    k8s-app: kubernetes-dashboard
 ##### end #####
# 创建service account并绑定默认cluster-admin管理员集群角色：
kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
# 使用输出的token登录Dashboard
  
kubectl create serviceaccount k8s-admin -n kube-system
kubectl create clusterrolebinding k8s-admin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-admin
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/k8s-admin/{print $1}')
 
# check cluster status
kubectl get nodes
NAME                         STATUS   ROLES    AGE   VERSION
node1  			      Ready    <none>   22h   v1.17.4
master                        Ready    master   22h   v1.17.4
node2                         Ready    <none>   22h   v1.17.4
```