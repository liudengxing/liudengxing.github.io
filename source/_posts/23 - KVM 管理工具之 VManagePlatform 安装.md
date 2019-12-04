---
title: KVM 管理工具之 VManagePlatform 安装
thumbnail: 'https://piccdn.freejishu.com/images/2019/07/31/PnYYoY.jpg'
abbrlink: c1324beb
date: 2019-07-30 16:50:31
tags:
categories:
---

*这是六等星的小宇宙的第 23 篇文章*



KVM管理工具虽然种类繁多，但开源的，国人制作的极少，这次要介绍的就是我认为还 ‘ 差一点 ’ 完美的 KVM 开源 WEB 管理工具 —— [VManagePlatform](https://github.com/welliamcao/VManagePlatform) 。



VManagePlatform 是一个 KVM 虚拟化管理平台，由 Python 开发，使用的框架有 Bootstrap 、Django 、Celery 和 Redis 。可以实现例如 资源调控 、 实例控制 、用户管理 等功能 。



VManagePlatform 管理主机和主机节点之间通过 TCP 协议通讯 ，节点的 安装是管理主机安装流程的弱化版 ，在下面所示的安装过程中 ，控制主机需要执行全部步骤，节点只需执行 2 、3 、4 步骤 ，以及步骤 5 的 SSH 信赖添加 。



那就让我们开始吧 。本文的安装方法 80% 源于官方文档 ， 20% 为自己修改 。



#### 一 、 配置需求模块

```
# yum install zlib zlib-devel readline-devel bzip2-devel openssl-devel gdbm-devel libdbi-devel ncurses-libs kernel-devel libxslt-devel libffi-devel python-devel libvirt libvirt-client libvirt-devel gcc git mysql-devel -y
# mkdir -p /opt/apps/ && cd /opt/apps/
# git clone https://github.com/welliamcao/VManagePlatform.git
# cd VManagePlatform
# pip install -r requirements.txt
```



#### 二 、安装 KVM

```
1、关闭防火墙 ，selinux
# systemctl stop firewalld.service && systemctl disable firewalld.service
# setenforce 0 临时关闭
# vim /etc/selinux/config 改为 disable 永久关闭
# systemctl stop NetworkManager && systemctl disable NetworkManager

2、安装 kvm 虚拟机
# yum install python-virtinst qemu-kvm virt-viewer bridge-utils virt-top libguestfs-tools ca-certificates libxml2-python audit-libs-python device-mapper-libs 
## 启动服务
# systemctl start libvirtd

注：下载 virtio-win-1.5.2-1.el6.noarch.rpm ，如果不安装 window 虚拟机或者使用带 virtio 驱动的镜像可以不用安装 virtio
# rpm -ivh virtio-win-1.5.2-1.el6.noarch.rpm

节点服务器不必执行
# yum -y install dnsmasq
# mkdir -p /var/run/dnsmasq/
```



#### 三 、安装 OpenVswitch （ 若是使用 Linux Bridge 作为底层网络可不必安装 ）

```
安装 openvswitch
# yum install gcc make python-devel openssl-devel kernel-devel graphviz kernel-debug-devel autoconf automake rpm-build redhat-rpm-config libtool 
# cd /usr/local/src
# wget http://openvswitch.org/releases/openvswitch-2.3.1.tar.gz
# tar xfz openvswitch-2.3.1.tar.gz
# mkdir -p ~/rpmbuild/SOURCES
# cp openvswitch-2.3.1.tar.gz ~/rpmbuild/SOURCES
# sed 's/openvswitch-kmod, //g' openvswitch-2.3.1/rhel/openvswitch.spec > openvswitch-2.3.1/rhel/openvswitch_no_kmod.spec
# rpmbuild -bb --without check openvswitch-2.3.1/rhel/openvswitch_no_kmod.spec
# yum localinstall ~/rpmbuild/RPMS/x86_64/openvswitch-2.3.1-1.x86_64.rpm
如果出现 python 依赖错误
# vim openvswitch-2.3.1/rhel/openvswitch_no_kmod.spec
BuildRequires: openssl-devel
后面添加
AutoReq: no

启动 openvswitch
# systemctl start openvswitch
```



#### 四 、配置 Libvirt 使用 tcp 方式连接

```
# vim /etc/sysconfig/libvirtd
LIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf
LIBVIRTD_ARGS="--listen"

# vim /etc/libvirt/libvirtd.conf #最后添加
  listen_tls = 0
  listen_tcp = 1
  tcp_port = "16509"
  listen_addr = "0.0.0.0"
  auth_tcp = "none"
# systemctl restart libvirtd 
```



#### 五 、配置SSH信任

```
# ssh-keygen -t  rsa #生成SSH证书
# ssh-copy-id -i ~/.ssh/id_rsa.pub  root@ipaddress #将SSH证书传递到目标主机上 ，实现SSH免密连接
```



#### 六 、安装数据库 （ MySQL ，Redis ）

参考我的两篇文章

[CentOS 离线安装 MySQL](https://nightmare233.top/posts/8d1db23b.html)

[CentOS 安装 Redis5.0.3](https://nightmare233.top/posts/4c767999.html)



#### 七 、配置 Django

```
# cd /opt/apps/VManagePlatform/VManagePlatform/
# vim settings.py
7.1、修改BROKER_URL：改为自己的地址
7.2、修改DATABASES：
DATABASES = {
    'default': {
        'ENGINE':'django.db.backends.mysql',
        'NAME':'vmanage',
        'USER':'自己的设置的账户',
        'PASSWORD':'自己的设置的密码',
        'HOST':'MySQL地址'
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
7.3、修改STATICFILES_DIRS
STATICFILES_DIRS = (
     '/opt/apps/VManagePlatform/VManagePlatform/static/',
    )
TEMPLATE_DIRS = (
#     os.path.join(BASE_DIR,'mysite\templates'),
    '/opt/apps/VManagePlatform/VManagePlatform/templates',
)
```



#### 八 、生成 VManagePlatform 数据表

```
# cd /opt/apps/VManagePlatform/
# python manage.py migrate
# python manage.py createsuperuser
```



#### 九 、启动 VManagePlatform

```
# cd /opt/apps/VManagePlatform/
# python manage.py runserver youripaddr:8000 #注意修改为你的IP地址
```



#### 十 、配置任务系统

```
# echo_supervisord_conf > /etc/supervisord.conf
# vim /etc/supervisord.conf
最后添加
[program:celery-worker]
command=/usr/bin/python manage.py celery worker --loglevel=info -E -B  -c 2
directory=/opt/apps/VManagePlatform
stdout_logfile=/var/log/celery-worker.log
autostart=true
autorestart=true
redirect_stderr=true
stopsignal=QUIT
numprocs=1

[program:celery-beat]
command=/usr/bin/python manage.py celery beat
directory=/opt/apps/VManagePlatform
stdout_logfile=/var/log/celery-beat.log
autostart=true
autorestart=true
redirect_stderr=true
stopsignal=QUIT
numprocs=1

[program:celery-cam]
command=/usr/bin/python manage.py celerycam
directory=/opt/apps/VManagePlatform
stdout_logfile=/var/log/celery-celerycam.log
autostart=true
autorestart=true
redirect_stderr=true
stopsignal=QUIT
numprocs=1

启动 celery 
# supervisord -c /etc/supervisord.conf #这里和官方文档不同，我是直接使用的 supervisord 启动
# supervisorctl status
```



OK ，搞定！

接下来就是访问 IP:8000 来使用VManagePlatform 了 。

如果你觉得这个项目不错 ，且对你有帮助 ，不妨前往作者的 Github 点个 Star ，为开源社区贡献一点自己的力量 。