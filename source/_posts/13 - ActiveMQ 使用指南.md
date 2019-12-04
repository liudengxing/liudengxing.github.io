---
title: ActiveMQ使用指南
tags:
  - 物联网
  - MQTT
  - ActiveMQ
categories:
  - 物联网
thumbnail: https://piccdn.freejishu.com/images/2019/04/19/Pnxpr5.jpg
abbrlink: 32500dbd
date: 2019-04-17 09:46:18
---
# ActiveMQ使用指南
*这是六等星的小宇宙的第 13 篇文章  
写作时间 : 1小时30分钟
总字数 : 2311字   
预计阅读时间 : 3分钟*

### 什么是ActiveMQ

>Apache ActiveMQ™ is the most popular open source, multi-protocol, Java-based messaging server. It supports industry standard protocols so users get the benefits of client choices across a broad range of languages and platforms. Connectivity from C, C++, Python, .Net, and more is available. Integrate your multi-platform applications using the ubiquitous AMQP protocol. Exchange messages between your web applications using STOMP over websockets. Manage your IoT devices using MQTT. Support your existing JMS infrastructure and beyond. ActiveMQ offers the power and flexibility to support any messaging use-case.

“Apache ActiveMQ”是最受欢迎的开源、多协议、依赖于java的信息服务器。ActiveMQ提供行业标准协议以便用户可以在大范围的语言与平台上使用，包括 C, C++, Python, .Net等。你可以使用无处不在的AMQP协议来整合你的多平台应用；使用依赖于websockets的STOMP协议来交换你的web应用间的数据；使用MQTT管理你的IoT设备。ActiveMQ还支持你现有的JMS架构以及更多的架构。总而言之，ActiveMQ提供有力又不失灵活的信息服务支持。  

### ActiveMQ特性  
- 多语音与协议编写客户端。包含Java，C，C++，Python等，支持协议:OpenWire,Stomp REST,WS Notification等
- 完全支持JM1.1与J2EE 1.4规范（持久化，XA消息，事务）
- 对Spring的支持，可内嵌使用，支持Spring2.0特性
- 通过了常见J2EE服务器(如 Geronimo,JBoss 4, GlassFish,WebLogic)的测试,其中通过JCA 1.5 resource adaptors的配置,可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上
- 支持多种传送协议:in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA
- 支持通过JDBC和journal提供高速的消息持久化
- 从设计上保证了高性能的集群,客户端-服务器,点对点
- 支持Ajax 支持与Axis的整合
- 可以很容易得调用内嵌JMS provider,进行测试

### 我们什么时候使用ActiveMQ
多个项目之间集成  
- 跨平台
- 多语言
- 多项目  

降低系统间模块的耦合度，解耦  
- 软件扩展性   

系统前后端隔离  
- 前后端隔离，屏蔽高安全区

### 安装ActiveMQ  
我使用的环境是Windows10，这里就以win10来进行说明   

首先前往[ActiveMQ官网](https://activemq.apache.org/index.html)进行下载  

官网提供两个版本，分别为"Classic"与“Artemis”  
“Classic”可以看做是“稳定版”，“Artemis”则是“开发版”。Classic最新版本号为5，而Artemis迭代足够多的特性后就会转为Classic 6，本篇就以Classic 5 作为演示版本。  
我使用的具体版本为apache-activemq-5.15.9

### 启动ActiveMQ
在路径apache-activemq-5.15.9/bin下可以看到win32与win64两个文件夹，分别对应32位与64位系统  
![ActiveMQ运行目录](https://piccdn.freejishu.com/images/2019/04/21/PnxfVT.png)    
我使用得是64位的win10，就打开win64，找到activemq.bat，这就是ActiveMQ的启动脚本  
![Active启动目录](https://piccdn.freejishu.com/images/2019/04/21/PnxYog.png)    
双击运行   
![ActiveMQ启动](https://piccdn.freejishu.com/images/2019/04/21/PnxVPc.png)     
运行成功后，AcitveMQ默认端口为8161，我们访问[http://localhost:8161/admin](http://localhost:8161/admin) 来访问ActiveMQ控制台。此时网页要求输入账号和密码，默认账号密码都是admin，可以在apache-activemq-5.15.9/conf下的users.properties自行配置。  
此时我们就可以看见ActiveMQ的控制台了。
![ActiveMQ控制台](https://piccdn.freejishu.com/images/2019/04/21/PnxhyI.png)
### 关闭ActiveMQ
在Windows下关闭命令行窗口即可。  

参考阅读  
[ActiveMQ的简单使用](https://nightmare233.top/posts/4839de24.html)
