---
title: 在CentOS7上安装Docker
tags:
  - Docker
categories:
  - Docker
thumbnail: 'https://piccdn.freejishu.com/images/2019/05/22/PnVOq3.jpg'
abbrlink: 272161d4
date: 2019-05-22 11:33:05
---
# 在CentOS7上安装Docker

Docker分为CE社区版与EE企业版，我们需要的是CE版本。

CE版Docker按stable与edge两种方式发布更新，edge每月更新而stable每季度更新。

#### 安装Docker

安装Docker需要CentOS内核版本高于3.10

##### 查看当前内核版本，确认能否使用Docker

```
uname -r
```

##### 更新

```
yum update
```

##### 卸载旧的Docker版本

```
yum remove docker  docker-common docker-selinux docker-engine
```

##### 安装需求环境

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

yum-utils提供了yum的yum-config-manager

device-mapper-persistent-data与lvm2是devicemapper驱动依赖

##### 设置yum源

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

因为某些404的原因，直接访问官方源的速度极慢极慢，所以这里用的是阿里云的国内镜像源。

##### 安装Docker

```
sudo yum install docker-ce
```

##### 启动Docker

```
systemctl start docker
```

##### 验证是否安装成功

```
docker version
```

##### 将Docker加入开机启动

```
systemctl enable docker
```
