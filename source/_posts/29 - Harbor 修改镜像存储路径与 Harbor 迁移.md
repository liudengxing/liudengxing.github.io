---
title: Harbor 修改镜像存储路径与 Harbor 迁移
thumbnail: 'https://piccdn.freejishu.com/images/2019/08/02/PnY5R4.jpg'
abbrlink: c4d8997c
date: 2019-08-02 09:56:00
tags:
categories:
---

*这是 六等星 的第 29 篇文章*



在 Harbor 安装好后想要修改 Harbor 镜像安装地址的方法如下 ：

首先关闭 Harbor ：

```
docker-compose down -v
```

修改 Harbor 配置文件 harbor.yml ，找到下面的位置 ，data_volume 后面跟着的既是 Harbor 的镜像存储地址 ，在这里的是我已经修改过了的 ， 默认是 /data 。

```
# The default data volume
data_volume: /home/docker_images
```

修改好后运行 Harbor 目录下的 prepare 文件 ：

```
./prepare
```

再启动 Harbor 即可 ：

```
docker-compose up -d
```

不过这个改法存在一个问题 ，即你的 Harbor 原有的镜像和配置等会全部丢失 。

那么丢失的镜像与配置在哪里 ？就在原本的目录下 ，有 ca_download 、database 、job_logs 、psc 、redis 、registry 、secret 共计六个目录 。将这六个目录打包放到新的存储路径下 ，丢失的数据与镜像就都回来啦 。

搞定 ！

这就是 Harbor 迁移与镜像存储的全部内容啦 。