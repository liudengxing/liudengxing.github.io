---
title: CentOS 安装 Redis5.0.3
abbrlink: 4c767999
date: 2019-07-31 09:15:36
tags:
categories:
thumbnail: https://piccdn.freejishu.com/images/2019/07/31/PnYjsH.jpg
---

*这是六等星的小宇宙的第 25 篇文章*



Redis 是我们常用的数据库之一 ，安装也是我们经常做的操作 ，那我们看看怎么安装他吧 。



下载 redis （ 这里使用的是 redis 5.0.3 ，目前最新版本为 5.0.5 ，但我下载几次都没成功 ，只有 5.0.3 版本能稳定下载 ）:

```
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
```

解压 redis :

```
tar -zxf redis-5.0.3.tar.gz
```

进入 redis 目录 :

```
cd redis-5.0.3
```

make :

```
make MALLOC=libc
```

不添加 MALLOC = libc 会报 jemalloc/jemalloc.h 目录不存在的错误 。

测试redis是否安装成功 ：

```
[root@localhost redis-5.0.3]# src/redis-server 
18905:C 03 Jun 2019 15:48:12.631 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
18905:C 03 Jun 2019 15:48:12.631 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=18905, just started
18905:C 03 Jun 2019 15:48:12.631 # Warning: no config file specified, using the default config. In order to specify a config file use src/redis-server /path/to/redis.conf
18905:M 03 Jun 2019 15:48:12.632 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 18905
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

18905:M 03 Jun 2019 15:48:12.634 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
18905:M 03 Jun 2019 15:48:12.635 # Server initialized
18905:M 03 Jun 2019 15:48:12.635 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
18905:M 03 Jun 2019 15:48:12.635 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
18905:M 03 Jun 2019 15:48:12.635 * Ready to accept connections
```

修改配置文件redis.conf ：

```
vim redis.conf
```

要修改的参数有三个 ：

```
daemonize no 改为 yes 后台运行：

protected-mode yes 改为no 可以不用输入密码登陆

bind 127.0.0.1  表示只可以本机访问，要是远程访问需要注释掉（前面加#号）
```

带配置文件后台启动 ：

```
[root@localhost redis-5.0.3]# src/redis-server redis.conf
18912:C 03 Jun 2019 15:51:47.863 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
18912:C 03 Jun 2019 15:51:47.863 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=18912, just started
18912:C 03 Jun 2019 15:51:47.863 # Configuration loaded
```

查看是否启动成功 ：

```
[root@localhost redis-5.0.3]# netstat -nlpt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      18913/src/redis-ser 
```

看到6379端口的服务就代表启动成功 。

