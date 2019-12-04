---
title: CoAP的简单使用
tags:
  - 物联网
  - CoAP
categories:
  - 物联网
thumbnail: 'https://piccdn.freejishu.com/images/2019/04/22/PnxNyr.jpg'
abbrlink: cab443d3
date: 2019-04-22 14:32:22
---
*这是六等星的小宇宙的第 16 篇文章   
写作时间 :  15分钟  
总字数 :  2816字  
预计阅读时间 :  3分钟*

在[前面的文章](https://nightmare233.top/posts/983c576.html)中，我简单介绍了CoAP协议的大概情况,今天我们就来看看CoAP的具体使用。  
- 本项目使用californium作为CoAP服务器  

我们首先在IDEA中新建一个Maven项目，然后根据californium[GitHub仓库](https://github.com/eclipse/californium)说明文档进行maven的配置，具体配置如下:  
```  
  <dependencies>
      <dependency>
          <groupId>org.eclipse.californium</groupId>
          <artifactId>californium-core</artifactId>
          <version>1.0.7</version>
      </dependency>
      <dependency>
          <groupId>org.eclipse.californium</groupId>
          <artifactId>element-connector</artifactId>
          <version>1.0.7</version>
      </dependency>
      <dependency>
          <groupId>org.eclipse.californium</groupId>
          <artifactId>scandium</artifactId>
          <version>1.0.7</version>
      </dependency>
  </dependencies>
```    

之后，我们新建一个CoAP发送端  

```  
import org.eclipse.californium.core.CoapResource;
import org.eclipse.californium.core.CoapServer;
import org.eclipse.californium.core.coap.CoAP;
import org.eclipse.californium.core.server.resources.CoapExchange;

import java.text.SimpleDateFormat;
import java.util.Date;

public class CoAPServer {
    public static void main(String[] args){
        //创建CoAP服务器
        CoapServer server = new CoapServer();

        server.add(new CoapResource("hello"){

            @Override
            public void handleGET(CoapExchange exchange) {
                exchange.respond(CoAP.ResponseCode.CONTENT,"Hello");
            }
        });

        server.add(new CoapResource("time"){
            @Override
            public void handleGET(CoapExchange exchange) {
                Date date = new Date();
                exchange.respond(CoAP.ResponseCode.CONTENT,new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(date));
            }
        });

        //启动服务器
        server.start();
    }
}
```  

再新建一个接收端

```  
import org.eclipse.californium.core.*;
import java.net.URI;
import java.net.URISyntaxException;

public class CoAPClient {

    public static void main(String[] args) throws URISyntaxException{
        URI uri = null;
        //设定请求路径
        uri = new URI("localhost:5683/hello");
        //创建客户端
        CoapClient client = new CoapClient(uri);
        //发起一个GET请求
        CoapResponse response = client.get();
        if(response != null){
            System.out.println(response.getCode());
            System.out.println(response.getOptions());
            System.out.println(response.getResponseText());
            System.out.println("\nAdvanced\n");
            System.out.println(Utils.prettyPrint(response));
        }
    }
}
```  

项目的主体框架就搭好了，我们首先运行服务端CoAPServer.main，控制台输出如下:  
![CoAP服务端控制台输出](https://piccdn.freejishu.com/images/2019/04/22/PnxZTu.png)  
这时我们再运行客户端CoAPClient.main，控制台输出如下:  
![CoAP客户端控制台输出](https://piccdn.freejishu.com/images/2019/04/22/PnxUGa.png)  
可以看到，消息已成功传递。
