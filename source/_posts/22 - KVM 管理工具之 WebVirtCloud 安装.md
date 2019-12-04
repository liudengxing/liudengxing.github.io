---
title: KVM 管理工具之 WebVirtCloud 安装
thumbnail: 'https://piccdn.freejishu.com/images/2019/07/30/PnYVT0.jpg'
abbrlink: faae2995
date: 2019-07-30 14:02:11
tags:
categories:
---

*这是六等星的小宇宙的第 22 篇文章*

KVM 是我们在公司中不可避免会用到的虚拟化技术 ，但是纯命令行Web页面的管理方法不够方便 ，不够“直觉”。所以就会有各种各样的 WEB 管理工具的出现，例如 OpenStack 、Pronebula 、oVirt 、WebVirtMgr 等 ，这次要介绍的就是 WebVirtMgr 的升级版本 [WebVirtCloud](https://github.com/retspen/webvirtcloud) 。



WebVirtCloud 相比于 WebVirtMgr 的改变在于添加了用户管理，页面变更等，安装方式和配置管理在很多地方是相似的，如果有疑问的地方不妨去看看 WebVirtMgr 的解决方法 。



那么接下来进入正篇，如何安装 WebVirtCloud ，安装环境为 CentOS7 ，系统纯净 ，安装方法 80% 参考官方文档 ，20% 源于各个博客 。



#### 第一步 安装依赖


先安装 wget ，获取 eprl 源 ：

  ```
# yum -y install wget
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
  ```

  然后再安装环境依赖 ：

```
# sudo yum -y install python-virtualenv python-devel libvirt-devel glibc gcc nginx supervisor python-lxml git python-libguestfs python-pip policycoreutils-python vim
```



#### 第二步 Clone 项目到本地

 在根目录下创建目录 /srv ：

```
# sudo mkdir /srv && cd /srv
```

Clone 项目到本地 ：

```
# git clone https://github.com/retspen/webvirtcloud && cd webvirtcloud
```

将配置文件模板复制一份作为设置 ：

```
# cp webvirtcloud/settings.py.template webvirtcloud/settings.py
```

现在设置密匙，在这里我并没有生成一个密匙，而是偷懒自己设置了一个密匙，有需要的可以参考官方文档设置密匙 ：

```
## now put secret key to webvirtcloud/settings.py

# sed -i "s#SECRET_KEY = ''#SECRET_KEY = 'abcd12345'#g" /srv/webvirtcloud/webvirtcloud/settings.py
```



#### 开始安装 WebVirtCloud

```
# virtualenv venv
# source venv/bin/activate
# venv/bin/pip install -r conf/requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple #这里使用了清华源
# cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d/
# venv/bin/python manage.py migrate
```



#### 配置 CentOS 上的 supervisor

修改配置文件 /etc/supervisord.conf ，在文件最末尾，也就是  [include] line (after files = ...  actually); 之后 ：

```
[program:webvirtcloud]
command=/srv/webvirtcloud/venv/bin/gunicorn webvirtcloud.wsgi:application -c /srv/webvirtcloud/gunicorn.conf.py
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true

[program:novncd]
command=/srv/webvirtcloud/venv/bin/python /srv/webvirtcloud/console/novncd
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true
```



#### 配置 Nginx 配置文件

修改配置文件 /etc/nginx/nginx.conf ，参照如下样本修改 server 设置 ：

```
#    server {
#        listen       80 default_server;
#        listen       [::]:80 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
#}
```

同时修改  /etc/nginx/conf.d/webvirtcloud.conf 配置文件 ：

```
upstream gunicorn_server {
    #server unix:/srv/webvirtcloud/venv/wvcloud.socket fail_timeout=0;
    server 127.0.0.1:8000 fail_timeout=0;
}
server {
    listen 80;

    server_name servername.domain.com;
    access_log /var/log/nginx/webvirtcloud-access_log; 

    location /static/ {
        root /srv/webvirtcloud;
        expires max;
    }
:：
    location / {
        proxy_pass http://gunicorn_server;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M;
    }
}
```

给予 nginx 读取 webvirtcloud 文件夹的权限 ：

```
# chown -R nginx:nginx /srv/webvirtcloud
```

修改selinux权限 ：

```
# semanage fcontext -a -t httpd_sys_content_t "/srv/webvirtcloud(/.*)"
```

创建新的管理用户并将用户加入 kvm 组 ：

```
# useradd webvirtcloud
# echo "123456" | passwd --stdin webvirtcloud
# usermod -G kvm -a webvirtcloud
```



#### 安装 libvirtd 以及其他主机的环境需求

```
# wget -O - https://clck.ru/9V9fH | sudo sh
```

开启服务

```
# systemctl restart nginx && systemctl restart supervisord
```

最终确定服务是否运行正常

```
# supervisorctl status
gstfsd             RUNNING   pid 24662, uptime 6:01:40
novncd             RUNNING   pid 24661, uptime 6:01:40
webvirtcloud       RUNNING   pid 24660, uptime 6:01:40
```



##### 搞定！

访问 http://serverip 来使用 WebVirtCloud 服务 。

如果不能访问就查看下 firewall 与 selinux 配置 。