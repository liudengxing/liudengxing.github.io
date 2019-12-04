---
title: Centos7 安装 Docker
thumbnail: 'https://piccdn.freejishu.com/images/2019/08/01/PnYBQh.jpg'
abbrlink: 69d36aac
date: 2019-08-01 09:36:22
tags:
categories:
---

*这是六等星的第 26 篇文章*



 Docker 方式的虚拟化在这几年异军突起 ，也逐渐被各个公司所接受并使用 ，我们就来看看 Docker 该怎么安装吧 。

首先我们来选择 Docker 发行版本 ，Docker 分为 CE 社区版与 EE 企业版，我们选择的是 CE 版本。

CE 版 Docker 分为 stable 与 edge 两种方式发布更新 ，edge 每月更新 、stable 每季度更新。



#### 安装Docker依赖环境

Docker 要求 CentOS 内核版本高于 3.10 ，首先查看当前内核版本，确认能否使用 ：

```
# uname -r
3.10.0-957.el7.x86_64
```



更新系统软件 ：

```
yum update
```



如果之前安装过 Docker 的话还需要卸载旧的 Docker ：

```
yum remove docker  docker-common docker-selinux docker-engine
```



安装 Docker 环境 ：

```
# yum install -y yum-utils device-mapper-persistent-data lvm2
...

已安装:
  yum-utils.noarch 0:1.1.31-50.el7                                                          

作为依赖被安装:
  libxml2-python.x86_64 0:2.9.1-6.el7_2.3       python-chardet.noarch 0:2.2.1-1.el7_1      
  python-kitchen.noarch 0:1.1.1-5.el7          

完毕！
```

- yum-utils 提供了 yum 的 yum-config-manager
- device-mapper-persistent-data 与 lvm2 是 devicemapper 驱动依赖



设置 yum Docker 源为阿里云源 ：

```
[root@MiWiFi-R1CM-srv ~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

因为某些 404 的原因，直接访问官方源的速度极慢极慢，所以这里用的是阿里云的国内镜像源 。

附上官方源 ：

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```



#### 安装 Docker

安装 Docker 本体 ：

```
# yum -y install docker-ce
....

已安装:
  docker-ce.x86_64 3:18.09.6-3.el7                                                          

作为依赖被安装:
  audit-libs-python.x86_64 0:2.8.4-4.el7    checkpolicy.x86_64 0:2.5-8.el7                  
  container-selinux.noarch 2:2.95-2.el7_6   containerd.io.x86_64 0:1.2.5-3.1.el7            
  docker-ce-cli.x86_64 1:18.09.6-3.el7      libcgroup.x86_64 0:0.41-20.el7                  
  libsemanage-python.x86_64 0:2.5-14.el7    policycoreutils-python.x86_64 0:2.5-29.el7_6.1  
  python-IPy.noarch 0:0.75-6.el7            setools-libs.x86_64 0:3.3.8-4.el7   
```



启动 Docker ：

```
# systemctl start docker
```



验证是否安装成功 ：

```
# docker version

Client: Docker Engine - Community
 Version:           19.03.1
 API version:       1.40
 Go version:        go1.12.5
 Git commit:        74b1e89
 Built:             Thu Jul 25 21:21:07 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.5
  Git commit:       74b1e89
  Built:            Thu Jul 25 21:19:36 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```



将Docker加入开机启动 ：

```
# systemctl enable docker

Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```



搞定 ！

Docker 已经安装完成啦 ~