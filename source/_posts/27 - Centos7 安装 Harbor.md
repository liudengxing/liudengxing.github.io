---
title: Centos7 安装 Harbor
thumbnail: 'https://piccdn.freejishu.com/images/2019/08/01/PnYGCQ.jpg'
abbrlink: bcbaf749
date: 2019-08-01 09:36:27
tags:
categories:
---

*这是六等星的第 27 篇文章*



Docker 仓库的解决方法大体上有这几种 ：

- Registry

  Registry 是 Docker 官方提供的本地私有仓库服务镜像 。

- Portus

  基于 Docker Registry API （ V2 ） 的开源前端与授权工具，最低 registry 版本要求为 2.1 ，可以作为授权服务器与用户界面 。

- harbor

  Vmware 公司开发，包括了权限管理 ( RBAC ) 、 LDAP 、审计、管理界面、自我注册、HA 等企业必需的功能，同时针对中国用户的特点，设计镜像复制和中文支持等功能  。



Docker 私有仓库解决方案中最常用的一种就是使用 Harbor 来搭建企业级的私有仓库 。  



本文就来介绍如何安装 Harbor 来实现私有仓库 。  



首先需要确保服务器已经安装好 Docker 环境 ，然后开始安装流程 。  



#### 安装 Docker Compose



##### 方法一 通过二进制文件直接安装 （ 推荐 ）

下载好 docker-compose-Linux-x86_64 并将文件传到目标主机下 。



修改 docker-compose-Linux-x86_64 的名字为 docker-compose ：

```
# mv docker-compose-Linux-x86_64 docker-compose
```



将 docker-compose 放至 /usr/local/bin 目录下

```
mv docker-compose /usr/local/bin
```



给 Docker Compose 赋权 ：

```
# sudo chmod +x /usr/local/bin/docker-compose
```



检查是否安装成功 ：

```
# docker-compose --version

docker-compose version 1.24.0, build 0aa590
```



##### 方法二 通过 github 下载安装

从 github 上下载文件 ：

```
# sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```



给 Docker Compose 赋权 ：

```
# sudo chmod +x /usr/local/bin/docker-compose
```



检查是否安装成功 ：

```
# docker-compose --version

docker-compose version 1.24.0, build 0aa590
```



##### 方法三 通过 pip 安装



检查 pip 是否已被安装 ：

```
# pip -V
```



如果没有就安装 pip ：

```
# yum -y install epel-release
# yum install python-pip
```



对安装好的 pip 进行升级  ：

```
# pip install --upgrade pip
```



再次检查 ：

```
# pip -V
```



安装 Docker-Compose ：

```
# sudo pip install docker-compose
```



安装完成后查看 Docker-Compose 安装状态 ：

```
# docker-compose -version

docker-compose version 1.24.0, build 0aa590
```



#### 报错

```
 "Cannot uninstall '***'. It is a distutils installed project..."
```



#### 解决方法

```
sudo pip install *** --upgrade --ignore-installed ***
```



#### 报错

```
'module' object has no attribute 'GSSException'
```



#### 解决方法

```
vim /usr/lib/python2.7/site-packages/paramiko/ssh_gss.py

53,55行改成：
53 import gssapi.error
55 GSS_EXCEPTIONS = (gssapi.error.GSSException,)
```





#### 安装Harbor

在线安装包[下载地址](https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-online-installer-v1.8.0.tgz) 

离线安装包[下载地址](https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.0.tgz) 



解压 harbor 至 /usr/local ：

```
# tar xvf harbor-offline-installer-v1.8.0.tgz -C /usr/local
```



修改配置文件 ：

```
# vim /usr/local/harbor/harbor.yml

## 将 hostname: reg.mydomain.com 修改为 hostname: yourdomain
## 可以直接使用ip地址 ，就不需要配置 Host 解析
```



新增 Host 解析 yourdomain :

```
# vim /etc/hosts

ip yourdomain
```



#### 启动 Harbor 安装程序

这里因为网络问题选择了下载完整包在本地安装的

Harbor 自带的三个可选功能,有需要的话可以直接在安装的时候一起安装，也可以在后期有需要的时候再安装

- notary 

证书功能，实现 Harbot https 访问

- clair

镜像漏洞扫描工具，由 cereos 开发，可以检查镜像操作系统及以上安装包是否与一致的不安全包相匹配，从而提高镜像安全性。

- ChartMuseum

Kubernetes（ K8s ）包管理工具。能与各种流行 Kubernetes 环境与服务协同工作



启动安装程序 ：

```
# /usr/local/harbor/install.sh 


[Step 0]: checking installation environment ...

Note: docker version: 18.09.6

Note: docker-compose version: 1.24.0

[Step 1]: loading Harbor images ...
Loaded image: goharbor/harbor-log:v1.8.0
Loaded image: goharbor/harbor-db:v1.8.0
c88db349fb2f: Loading layer [==================================================>]  8.972MB/8.972MB
1f2d4d72bba2: Loading layer [==================================================>]  35.77MB/35.77MB
dddbcf598df5: Loading layer [==================================================>]  2.048kB/2.048kB
0ced476c2d9c: Loading layer [==================================================>]  3.072kB/3.072kB
af24eb0bf40b: Loading layer [==================================================>]  35.77MB/35.77MB
Loaded image: goharbor/chartmuseum-photon:v0.8.1-v1.8.0
Loaded image: goharbor/prepare:v1.8.0
257ebcc1c9c4: Loading layer [==================================================>]  8.967MB/8.967MB
7579d3c94fca: Loading layer [==================================================>]  38.68MB/38.68MB
323611f7dd17: Loading layer [==================================================>]  38.68MB/38.68MB
Loaded image: goharbor/harbor-jobservice:v1.8.0
587a5757a7f6: Loading layer [==================================================>]  3.548MB/3.548MB
Loaded image: goharbor/nginx-photon:v1.8.0
a61ab2060e6e: Loading layer [==================================================>]  8.967MB/8.967MB
25359ae00f57: Loading layer [==================================================>]  5.143MB/5.143MB
610a1668f8bf: Loading layer [==================================================>]  15.13MB/15.13MB
db2252abd9e0: Loading layer [==================================================>]  26.47MB/26.47MB
4f406312560b: Loading layer [==================================================>]  22.02kB/22.02kB
1cee0947e5a7: Loading layer [==================================================>]  3.072kB/3.072kB
48db2b9b0752: Loading layer [==================================================>]  46.74MB/46.74MB
Loaded image: goharbor/notary-server-photon:v0.6.1-v1.8.0
aaf447150765: Loading layer [==================================================>]    113MB/113MB
6835441e1a1d: Loading layer [==================================================>]  10.94MB/10.94MB
9f4739e3a532: Loading layer [==================================================>]  2.048kB/2.048kB
928f489135f0: Loading layer [==================================================>]  48.13kB/48.13kB
1495a1a09ada: Loading layer [==================================================>]  3.072kB/3.072kB
1a5f5b141717: Loading layer [==================================================>]  10.99MB/10.99MB
Loaded image: goharbor/clair-photon:v2.0.8-v1.8.0
66006ea937c6: Loading layer [==================================================>]  337.8MB/337.8MB
d272ba122880: Loading layer [==================================================>]  106.5kB/106.5kB
Loaded image: goharbor/harbor-migrator:v1.8.0
Loaded image: goharbor/harbor-core:v1.8.0
Loaded image: goharbor/redis-photon:v1.8.0
Loaded image: goharbor/registry-photon:v2.7.1-patch-2819-v1.8.0
Loaded image: goharbor/harbor-portal:v1.8.0
Loaded image: goharbor/harbor-registryctl:v1.8.0
453732ea69d4: Loading layer [==================================================>]  13.72MB/13.72MB
c985f3824f33: Loading layer [==================================================>]  26.47MB/26.47MB
76eaa2763221: Loading layer [==================================================>]  22.02kB/22.02kB
0ef55a752948: Loading layer [==================================================>]  3.072kB/3.072kB
c5749b90723d: Loading layer [==================================================>]  45.33MB/45.33MB
Loaded image: goharbor/notary-signer-photon:v0.6.1-v1.8.0


[Step 2]: preparing environment ...
prepare base dir is set to /usr/local/harbor
WARNING: IPv4 forwarding is disabled. Networking will not work.
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir



[Step 3]: starting Harbor ...
Creating harbor-log ... done
Creating registryctl ... done
Creating registry    ... done
Creating harbor-db   ... done
Creating redis       ... done
Creating harbor-core ... done
Creating harbor-jobservice ... done
Creating harbor-portal     ... done
Creating nginx             ... done

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.134.183 . 
For more details, please visit https://github.com/goharbor/harbor .
```



搞定 ！

现在就可以访问 http://IP 来使用 Harbor Web 管理页面了 。

如果不能访问就查看一下 firewalld 和 selinux 状态 。

```
# systemctl stop firewalld && systemctl disable firewalld
# setenforce 0

## 关闭防火墙后需要重新加载 docker ，否则会造成 docker chain 变动，无法启动 docker 容器
# systemctl restart docker 
```



#### 推送镜像

登录 Harbor ,默认用户名 admin ，默认密码 Harbor12345 ：

```
# docker login ip
```

如果出现 x509 证书认证的问题 ，就在 docker 的配置文件里添加 ip 访问信任 ：

```
# vim /etc/docker/daemon.json

{
  "insecure-registries" : ["ip"]
}

## 重启 docker 服务
systemctl daemon-reload 
systemctl restart docker
```

如果是搭建 Harbor 的主机第一次登录 Harbor 可能会出现 connect: connection refused 的问题 ，这时候需要先配置好 ip 访问信任 ，再重启 Harbor 服务 ，之后再次登录即可 ：

```
# cd /usr/local/harbor/
# docker-compose down -v
# docker-compose up -d
```



查看所有 Docker 镜像 ：

```
# docker images
```



可以选择下载一个 Hello-World 镜像作为测试

```
# docker pull hello-world
```

如果出现 net/http: TLS handshake timeout ，就前往[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)获取阿里云的 Docker 国内镜像加速服务 。

登录后在左边 -> 镜像加速器 ，将给出的加速器地址放到 /etc/docker/deamon.json 中即可 ，需要注意的是这会覆盖掉 "insecure-registries" : ["ip"] ，需要重新添加 。

```
# sudo mkdir -p /etc/docker
# sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://url.mirror.aliyuncs.com"],
  "insecure-registries" : ["ip"]
}
EOF
# sudo systemctl daemon-reload
# sudo systemctl restart docker
# docker-compose down -v
# docker-compose up -d
```



对要推送的镜像打上标签 ，标签可以在 Harbor 仓库页面查看 ：

```
# docker tag SOURCE_IMAGE[:TAG] yourdomain.com/library/IMAGE[:TAG]
```



推送镜像

```
# docker push yourdomain.com/library/IMAGE[:TAG]
```



这是推送成功的一个镜像信息

```
# docker push yourdomain.com.com/bilibili/bilibilihelper:latest

The push refers to repository [yourdomain.com/bilibili/bilibilihelper]
5f580f14341b: Pushed 
b93e6b2e9243: Pushed 
4da18bbd4cf1: Pushed 
c46a7044585c: Pushed 
8f8ce1ee4af3: Pushed 
4906c61c139f: Pushed 
a464c54f93a9: Pushed 
latest: digest: sha256:73c03641ad964b645d55d4142ca29b12d13f50332e7b4c6bd72284e733677af7 size: 1784
```

