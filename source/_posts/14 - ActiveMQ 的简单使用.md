---
title: ActiveMQ的简单使用
abbrlink: 4839de24
date: 2019-04-17 11:13:08
tags:
- MQTT
- ActiveMQ
- 物联网
categories:
- 物联网
thumbnail: https://piccdn.freejishu.com/images/2019/04/19/PnxiAR.jpg
---
# ActiveMQ的简单使用
*这是六等星的小宇宙的第 14 篇文章  
写作时间 : 2小时  
总字数 : 4949字   
预计阅读时间 : 5分钟*


在IDEA中创建一个项目，导入我们下载的ActiveMQ根目录下的jar包  
![ActiveMQ_Jar包](https://piccdn.freejishu.com/images/2019/04/21/PnxxGE.png)  
不多BB，直接上代码  
创建发送者  

```  
  import javax.jms.Connection;
  import javax.jms.ConnectionFactory;
  import javax.jms.DeliveryMode;
  import javax.jms.Destination;
  import javax.jms.MessageProducer;
  import javax.jms.Session;
  import javax.jms.TextMessage;
  import org.apache.activemq.ActiveMQConnection;
  import org.apache.activemq.ActiveMQConnectionFactory;

  public class Producter {
    private static final int SEND_NUMBER = 5;
    private static final String[] msg ={"你好", "这里是", "发往", "ActiveMQ", "的测试消息" };

    public static void main(String[] args) {
        // ConnectionFactory ：连接工厂，JMS 用它创建连接
        ConnectionFactory connectionFactory;
        // Connection ：JMS 客户端到JMS Provider 的连接
        Connection connection = null;
        // Session： 一个发送或接收消息的线程
        Session session;
        // Destination ：消息的目的地;消息发送给谁.
        Destination destination;
        // MessageProducer：消息发送者
        MessageProducer producer;
        // TextMessage message;
        // 构造ConnectionFactory实例对象，此处采用ActiveMq的实现jar
        connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_USER,
                ActiveMQConnection.DEFAULT_PASSWORD,
                ActiveMQConnection.DEFAULT_BROKER_URL);
        try {
            // 构造从工厂得到连接对象
            connection = connectionFactory.createConnection();
            // 启动
            connection.start();
            // 获取操作连接
            session = connection.createSession(Boolean.TRUE,
                    Session.AUTO_ACKNOWLEDGE);
            //创建消息队列,消息队列一致的客户端可以收到信息
            destination = session.createQueue("FirstQueue");
            // 得到消息生成者
            producer = session.createProducer(destination);
            // 设置不持久化
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            // 构造消息
            sendMessage(session, producer, msg);
            session.commit();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != connection)
                    connection.close();
            } catch (Throwable ignore) {
            }
        }
    }

    public static void sendMessage(Session session, MessageProducer producer, String[] msg)
            throws Exception {
        int i = 1;
        for(String msg1 : msg){
                TextMessage message = session.createTextMessage(msg1);
                System.out.println("已发送消息" + i++);
                // 发送消息
                producer.send(message);
        }
    }
  }
```  

创建接收者

```  
  import javax.jms.Connection;
  import javax.jms.ConnectionFactory;
  import javax.jms.Destination;
  import javax.jms.MessageConsumer;
  import javax.jms.Session;
  import javax.jms.TextMessage;
  import org.apache.activemq.ActiveMQConnection;
  import org.apache.activemq.ActiveMQConnectionFactory;

  public class Receiver {
      public static void main(String[] args) {
          // ConnectionFactory ：连接工厂，JMS 用它创建连接
          ConnectionFactory connectionFactory;
          // Connection ：JMS 客户端到JMS Provider 的连接
          Connection connection = null;
          // Session： 一个发送或接收消息的线程
          Session session;
          // Destination ：消息的目的地;消息发送给谁.
          Destination destination;
          // 消息接收者
          MessageConsumer consumer;
          connectionFactory = new ActiveMQConnectionFactory(
                  ActiveMQConnection.DEFAULT_USER,
                  ActiveMQConnection.DEFAULT_PASSWORD,
                  ActiveMQConnection.DEFAULT_BROKER_URL);
          try {
              // 构造从工厂得到连接对象
              connection = connectionFactory.createConnection();
              // 启动
              connection.start();
              // 获取操作连接
              session = connection.createSession(Boolean.FALSE,
                      Session.AUTO_ACKNOWLEDGE);
              destination = session.createQueue("FirstQueue");
              consumer = session.createConsumer(destination);
              while (true) {
                  //设置接收者接收消息的时间，单位为ms
                  TextMessage message = (TextMessage) consumer.receive(100000);
                  if (null != message) {
                      System.out.println("收到消息: " + message.getText());
                  } else {
                      break;
                  }
              }
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              try {
                  if (null != connection)
                      connection.close();
              } catch (Throwable ignore) {
              }
          }
      }
  }
```  

运行结果  
发送者控制台输出  
![发送者控制台输出](https://piccdn.freejishu.com/images/2019/04/21/PnxCXz.png)   
接收者控制台输出  
![接收者控制台输出](https://piccdn.freejishu.com/images/2019/04/21/Pnx444.png)   
ActiveMQ控制台结果  
![ActiveMQ控制台结果](https://piccdn.freejishu.com/images/2019/04/21/PnxeMU.png)
