---
title: CentOS 离线安装 MySQL
abbrlink: 8d1db23b
date: 2019-07-31 08:50:15
tags:
categories:
thumbnail: https://piccdn.freejishu.com/images/2019/07/31/PnYfV3.jpg
---

*这是六等星的小宇宙的第 24 篇文章*



在 CentOS 默认源中不包含 mysql 之后，在线安装 mysql 就变得不稳定，并且在线安装并不适合大批量安装，所以我就研究了 mysql 的离线安装技巧 。

首先在 mysql 官网上下载最新的 mysql 安装包，我是用的是 8.0.16 版本 。

共需要四个文件 ：

- mysql-community-common-8.0.16-2.el7.x86_64.rpm
- mysql-community-libs-compat-8.0.16-2.el7.x86_64.rpm
- mysql-community-client-8.0.16-2.el7.x86_64.rpm
- mysql-community-server-8.0.16-2.el7.x86_64.rpm

下好了就可以开始安装了 。



####  一 、卸载系统自带的 mariadb-libs 

查看 mariadb 包并写在 ：

```
rpm -qa|grep mariadb

rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```



#### 二 、安装 MySQL 

将下好的四个安装包上传到服务器后，推荐使用一键安装 ：

```
 rpm -Uvh *.rpm --nodeps --force
```

如果需要手动安装一定要按照我上面给出的顺序安装 ：

```
rpm -ivh mysql-community-common-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.16-2.el7.x86_64.rpm
```



安装完成后启动服务 ：

```
systemctl start mysqld.service
```

之后就可以愉快的使用了 ~



#### 三 、数据库初始化

数据库初始化 ：

```
mysqld --initialize --user=mysql
```

初始化会生成一个随机密码 ：

```
grep 'temporary password' /var/log/mysqld.log
```

复制这个随机码用于登录mysql ：

```
mysql -uroot -p
```

登录后设置新密码 ：

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
```

下次登录时使用新密码登录即可 。

完成 ！



#### 报错解决

##### 如果出现 ：

```
MySQL 8.0+ 密码问题 ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

那就是你的密码错了 ，或者你是空密码但是没有设置允许空密码登录 。

##### 解决方法 ：

先将 mysql 切到无需密码的登录模式 ：

```
vim /etc/my.cnf
```

在最后加上 ：

```
skip-grant-tables
```

重启 mysql 服务 ：

```
service mysqld restart
```

这时候就可以直接登陆了 ：

```
mysql -uroot -p
```

选择 mysql 数据库 ：

```
use mysql;
```

查看用户表 ：

```
select host,user,plugin,authentication_string from mysql.user;
```

可以看到 mysql8.0 之后都是使用 authentication_string 来保存密码，且加密格式为 caching_sha2_password  。

我们将 root 的密码设为空 ：

```
UPDATE mysql.user SET authentication_string='' WHERE user='root' and host='localhost';
```

退出 mysql ：

```
exit;
```

将上面加上的配置选项删去 。

重启 mysql 服务 ：

```
service mysqld restart
```

无密码登录 mysql ：

```
mysql -uroot -p
```

选择数据库 mysql ：

```
use mysql
```

修改 root 密码 ：

```
#caching_sha2_password 加密
ALTER user 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'root';
#mysql_native_password 加密
ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

退出mysql，之后使用新密码登录即可 。



##### 如果出现 ：

```
连接报 caching-sha2-password 验证错误
```

##### 解决方法 ：

登录 mysql ：

```
mysql -uroot -p
```

选择数据库 mysql ：

```
use mysql
```

修改 root 密码使用 mysql_native_password 加密  ：

```
#mysql_native_password 加密
ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

退出mysql，之后再次连接即可 。

