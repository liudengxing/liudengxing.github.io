---
title: Kubernetes 集群实现
thumbnail: 'https://piccdn.freejishu.com/images/2019/12/06/PnwTAu.jpg'
abbrlink: f4a00965
date: 2019-12-06 15:41:55
tags:
categories:
---

# Kubernetes 集群实现

### 1. 环境准备及介绍

#### 1.1 实现环境

使用 VMware 创建三台虚拟机 ，系统为CentOS 7 ，具体如下

```
[root@k8s-master ~]# uname -a
Linux k8s-master 3.10.0-1062.7.1.el7.x86_64 #1 SMP Mon Dec 2 17:33:29 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

三台主机分别配置了网络 ，具体如下 ： 

| 节点及功能             | 主机名     | IP              |
| ---------------------- | ---------- | --------------- |
| Master, etcd, registry | K8s-master | 192.168.134.181 |
| Node 1                 | K8s-node-1 | 192.168.134.183 |
| Node 2                 | K8s-node-2 | 192.168.134.184 |



设置三台主机名 : 

对 Master 执行 : 

```
[root@localhost ~]# hostnamectl --static set-hostname  k8s-master
```

对 Node 1 执行 : 

```
[root@localhost ~]# hostnamectl --static set-hostname  k8s-node-1
```

对 Node 2 执行 : 

```
[root@localhost ~]# hostnamectl --static set-hostname  k8s-node-2
```



设置三台主机的 Host :

```
echo '192.168.134.181    k8s-master
192.168.134.181   etcd
192.168.134.181   registry
192.168.134.183   k8s-node-1
192.168.134.184   k8s-node-2' >> /etc/hosts
```



关闭三台主机的防火墙

```
systemctl disable firewalld.service
systemctl stop firewalld.service
```



### 2. 安装 etcd

etcd 用于存储 k8s 的集群数据

```
[root@localhost ~]# yum -y install etcd
```

![img](https://piccdn.freejishu.com/images/2019/12/06/Pnw00r.png)

修改配置文件

```
[root@localhost ~]# vi /etc/etcd/etcd.conf

# [member]
ETCD_NAME=master  <- 修改为 Master
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
#ETCD_SNAPSHOT_COUNT="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"  <- 修改为新的端口
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#
#[cluster]
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd:2379,http://etcd:4001"  <- 修改为新的端口
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_SRV=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
```



启动并验证状态

```
[root@localhost ~]# systemctl start etcd
[root@localhost ~]#  etcdctl set testdir/testkey0 0
0
[root@localhost ~]#  etcdctl get testdir/testkey0 
0
[root@localhost ~]# etcdctl -C http://etcd:4001 cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://0.0.0.0:2379
cluster is healthy
[root@localhost ~]# etcdctl -C http://etcd:2379 cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://0.0.0.0:2379
cluster is healthy
```



### 3. 部署 Master



##### 安装 Kubernets

```
[root@k8s-master ~]# yum install kubernetes
```

安装 Kubernets 时会同时装上 Docker ，如果之前安装过 Docker 并在这一行命令执行时报依赖错误则执行

```
[root@k8s-master ~]# yum list installed | grep docker
[root@k8s-master ~]# yum remove -y docker-ce.x86_64
[root@k8s-master ~]# rm -rf /var/lib/docker
```



##### 配置 Docker

```
[root@k8s-master ~]# vim /etc/sysconfig/docker

# /etc/sysconfig/docker

# Modify these options if you want to change the way the docker daemon runs
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
OPTIONS='--insecure-registry registry:5000'  <- 添加一行
```

设置 Docker 开机自启

```
[root@k8s-master ~]# chkconfig docker on
[root@k8s-master ~]# service docker start
```



##### 配置 Kubernetes

Master 主机上需要运行 Kubernets API Server, Kubernets Controller Manager, Kubernets Scheduler

修改配置文件

```
[root@k8s-master ~]# vim /etc/kubernetes/apiserver

###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"  <- 修改

# The port on the local server to listen on.
KUBE_API_PORT="--port=8080"  <- 修改

# Port minions listen on
# KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://etcd:2379"  <- 修改

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# default admission control policies
#KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"  <- 修改

# Add your own!
KUBE_API_ARGS=""
```



```
[root@k8s-master ~]# vim /etc/kubernetes/config

###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://k8s-master:8080"  <- 修改
```



启动 Kubernetes 并设置开机自启

```
[root@k8s-master ~]# systemctl enable kube-apiserver.service
[root@k8s-master ~]# systemctl start kube-apiserver.service
[root@k8s-master ~]# systemctl enable kube-controller-manager.service
[root@k8s-master ~]# systemctl start kube-controller-manager.service
[root@k8s-master ~]# systemctl enable kube-scheduler.service
[root@k8s-master ~]# systemctl start kube-scheduler.service
```



### 4. 部署 node 主机



参照 Master 主机安装 Kubernets 和 Docker ，在 node 主机上需要运行的有 : Kublet, Kubernets Proxy ，修改配置文件 :

```
[root@K8s-node-1 ~]# vim /etc/kubernetes/config

###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://k8s-master:8080"  <- 修改
```



```
[root@K8s-node-1 ~]# vim /etc/kubernetes/kubelet

###
# kubernetes kubelet (minion) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"  <- 修改

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=k8s-node-1"  <- 修改

# location of the api-server
KUBELET_API_SERVER="--api-servers=http://k8s-master:8080"  <- 修改

# pod infrastructure container
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"

# Add your own!
KUBELET_ARGS=""
```



启动 node 主机服务并设置开机自启

```
[root@k8s-master ~]# systemctl enable kubelet.service
[root@k8s-master ~]# systemctl start kubelet.service
[root@k8s-master ~]# systemctl enable kube-proxy.service
[root@k8s-master ~]# systemctl start kube-proxy.service
```



#### 查看服务状态

在 Master 节点上此时可以查看鸡群节点和节点状态了

```
[root@k8s-master ~]# kubectl -s http://k8s-master:8080 get node
NAME         STATUS    AGE
k8s-node-1   Ready     8m
k8s-node-2   Ready     6m
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS    AGE
k8s-node-1   Ready     8m
k8s-node-2   Ready     6m
```

此时集群已经初步搭建完成 ，还需要 Flannel 来完成安装



### 创建覆盖网络 Flannel

在所有节点上安装 flannel

```
[root@k8s-master ~]# yum -y install flannel
```

![img](https://piccdn.freejishu.com/images/2019/12/06/PnwP6a.png)



修改配置

```
[root@k8s-master ~]# vi /etc/sysconfig/flanneld

# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD_ENDPOINTS="http://etcd:2379"  <- 修改

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/atomic.io/network"

# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
```



##### 配置 etcd 中 flannel 的 key

Flannel 使用 Etcd 进行配置，来保证多个 Flannel 实例之间的配置一致性，所以需要在 etcd 上进行如下配置：（‘/atomic.io/network/config’ 这个 key 与上文 /etc/sysconfig/flannel 中的配置项 FLANNEL_ETCD_PREFIX 是相对应的，错误的话启动就会出错）

```
[root@k8s-master ~]# etcdctl mk /atomic.io/network/config '{ "Network": "10.0.0.0/16" }'
{ "Network": "10.0.0.0/16" }
```



咱 Flannel 装完之后 ，大部分服务需要重新启动

Master 主机 :

```
systemctl enable flanneld.service 
systemctl start flanneld.service 
service docker restart
systemctl restart kube-apiserver.service
systemctl restart kube-controller-manager.service
systemctl restart kube-scheduler.service
```

Node 主机 : 

```
systemctl enable flanneld.service 
systemctl start flanneld.service 
service docker restart
systemctl restart kubelet.service
systemctl restart kube-proxy.service
```



搞定 ！