

# 安装启动RocketMQ

## RocketMQ介绍

RocektMQ是阿里巴巴在2012年开源的一个纯java、分布式消息中间件。RocektMQ是一款低延迟、高可靠、可伸缩、易于使用的消息中间件。2016年阿里巴巴将RocketMQ捐赠给Apache，2017年9月RocketMQ正式从Apache社区正式毕业，成为Apache顶级项目。

## 环境准备

+ CentOS 7
+ JDK 1.8+
+ Maven 3.2.x
+ Git

## 代码下载

```bash
 > git clone https://github.com/apache/rocketmq.git
```

## 项目构建

```bash
> cd rocketmq
# meavn打包
> mvn -Prelease-all -DskipTests clean install -U
# 这里的4.5.x是版本号，可能不同，请注意自己的版本
> cd distribution/target/rocketmq-4.5.x/rocketmq-4.5.x
```

## 开放端口号

```bash
#查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
$ firewall-cmd --state 
# 开启一个指定端口号
# --permanent 永久生效，没有此参数重启后失效 
$ firewall-cmd --zone=public --add-port=9876/tcp --permanent   
#重载防火墙规则
firewall-cmd --reload
```

## 修改RockMQ的JVM参数

```bash
# 修改NameServer的JVM
$ vi ./bin/runserver.sh 
# 将-Xms等值修改成如下
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

# 修改runbroker.sh的JVM参数
vim ./bin/runbroker.sh
# 将-Xms等值修改成如下
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m"
```

> 如果服务器的内存是8G以上, 就可以不用修改.

## 修改conf/broker.conf文件

```bash
vim ./conf/broker.conf

# 添加下面内容到最后
brokerIP1=你的服务器ip
namesrvAddr=你的服务器ip:9876
```

## 启动NameServer

```bash
# 启动NameServer
> nohup sh bin/mqnamesrv &
# 查看日志，确认是否成功
> tail -f ~/logs/rocketmqlogs/namesrv.log

2019-9-27 21:04:10 INFO NSScheduledThread1 - ----
...
#出现下面信息成功
  The Name Server boot success...
```

## 启动Broker

```bash
# 启动Broker
> nohup sh bin/mqbroker -n 81.68.182.114:9876 autoCreateTopicEnable=true -c conf/broker.conf &
# 查看日志，确认是否成功
> tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
```

> 注意：当无法启动的时候，可以使用sh命令，来看详细报错信息
>
> ```bash
> sh bin/mqnamesrv
> 
> sh bin/mqbroker -n 你的ip:9876
> ```

# 验证发送和接收消息

## 生产者发送消息

```bash
 > export NAMESRV_ADDR=你的ip:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 # 出现下面信息成功发送
  SendResult [sendStatus=SEND_OK, msgId= ...
```

## 消费者消费消息

```bash
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
# 出现下面信息成功消费
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

# 关闭RocketMQ服务

```bash
# 关闭 broker
$ sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

# 关闭 namesrv
$ sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

# RocketMQ-Console安装

## RocketMQ-Console介绍

RocketMQ有一个对其扩展的开源项目incubator-rocketmq-externals，这个项目中有一个子模块叫rocketmq-console，这个便是管理控制台项目了，提供了对RocketMQ的可视化管理工具，方便可视化的操作。
包含了多个功能：运维、驾驶舱、集群、主题、消费者、生产者、消息、消息轨迹、connector 等.

## 环境准备

+ Centos7
+ JDK1.8

## 安装

安装RocketMQ-Console，可以通过两种方式:

通过Docker镜像安装；

通过GitHub拉取源代码，进行编译，然后启动安装；

### 通过Docker方式

```bash
# 拉取镜像
# 还可以自己通过源码的方式打包镜像，需要有镜像仓库，想了解的可以在公号历史文章里面看一下docker相关的文章。打镜像命令：mvn clean package -Dmaven.test.skip=true docker:build
$ docker pull styletang/rocketmq-console-ng
# 启动,默认端口号是8080端口，映射到服务器里面可以按照需求进行更改
$ docker run -e "JAVA_OPTS=-Drocketmq.namesrv.addr=你的ip:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8080:8080 -t styletang/rocketmq-console-ng
```

### 通过源码方式

```bash
# 从GitHub上面拉取代码
$ git clone https://github.com/apache/rocketmq-externals.git
$ cd rocketmq-console
# maven打包
$ mvn clean package -Dmaven.test.skip=true
$ java -jar target/rocketmq-console-ng-2.0.0.jar
```

默认端口号是8080，可以到rocketmq-console/src/main/resources/application.properties进行修改。

```bash
server.address=0.0.0.0
# server.port=8080
# 我这里将8080改成19876了
server.port=19876
# …… 
# 中间省略
# ………
# 这里是指定Nameserv,也可以不指定，在前端控制台进行指定
rocketmq.config.namesrvAddr=localhost:9876
# …… 
# 后续省略
# ………
```

## 开放防火墙对应端口号

如果你的服务器开通了防火墙，需要对端口号进行开放

```bash
#查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
$ firewall-cmd --state 
# 开启一个指定端口号
# --permanent 永久生效，没有此参数重启后失效 
$ firewall-cmd --zone=public --add-port=19876/tcp --permanent   
#重载防火墙规则
firewall-cmd --reload
```

## 访问console页面

地址：http://localhost:8080(8080为docker映射端口.)

#  消息发送

## 主要内容

1. 基于Java环境构建消息发送与消息接受基础程序
   1. 单生产者单消费者
   2. 单生产者多消费者
   3. 多生产者多消费者
2. 发送不同类型的消息
   1. 同步消息
   2. 异步消息
   3. 单向消息
3. 特殊的消息发送
   1. 延时消息
   2. 批量消息
4. 消息发送与接收顺序控制
5. 事物消息

## 单生产者单消费者发送(OneToOne)

1. 新建maven项目rocketmq

2. 导入RocketMQ客户端坐标

   ```xml
           <dependency>
               <groupId>org.apache.rocketmq</groupId>
               <artifactId>rocketmq-client</artifactId>
               <version>4.8.0</version>
           </dependency>
   ```

3. 生产者

   ```java
   package xyz.rtx3090.first;
   
   import org.apache.rocketmq.client.producer.DefaultMQProducer;
   import org.apache.rocketmq.client.producer.SendResult;
   import org.apache.rocketmq.common.message.Message;
   
   
   public class HelloProducer {
       public static void main(String[] args) throws Exception {
           //1.创建一个发送消息的对象Producer
           DefaultMQProducer producer = new DefaultMQProducer("group1");
           //2.设定发送的命名服务器地址
           producer.setNamesrvAddr("81.68.182.114:9876");
           //3.1启动发送的服务
           producer.start();
           //4.创建要发送的消息对象,指定topic，指定内容body
           Message msg = new Message("topic1", "hello rocketmq".getBytes("UTF-8"));
           //3.2发送消息
           SendResult result = producer.send(msg);
           System.out.println("返回结果：" + result);
           //5.关闭连接
           producer.shutdown();
       }
   }
   ```

4. 消费者

   ```java
   package xyz.rtx3090.first;
   
   import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
   import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
   import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
   import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
   import org.apache.rocketmq.client.exception.MQClientException;
   import org.apache.rocketmq.common.message.MessageExt;
   
   import java.util.List;
   
   public class HelloConsumer {
       public static void main(String[] args) throws InterruptedException, MQClientException {
           // 1.创建一个接受消息Dee对象Consumer
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
           // 2.设定接收的命名服务器地址
           consumer.setNamesrvAddr("81.68.182.114:9876");
           // 3.设置接受消息对应的topic, 对应的sub标签为任意
           consumer.subscribe("topic1","*");
           // 4.开启监听, 用于接受消息
           consumer.registerMessageListener(new MessageListenerConcurrently() {
               @Override
               public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                   // 遍历消息
                   for (MessageExt msg: list) {
                       System.out.println("收到消息: " +msg);
                   }
                   return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
               }
           });
           // 5.启动接收消息的服务
           consumer.start();
           // 6.不要关闭消费者
       }
   }
   ```

5. 先启动"消费者", 然后再启动"生产者".

6. 查看rocketmq-console页面中发送的message

   ![image-20211221192320895](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211221192320895.png)

## 单生产者多消费者消息发送(OneToMany)

1. 生产者

   ```java
   package xyz.rtx3090.oneToMany;
   
   import org.apache.rocketmq.client.exception.MQBrokerException;
   import org.apache.rocketmq.client.exception.MQClientException;
   import org.apache.rocketmq.client.producer.DefaultMQProducer;
   import org.apache.rocketmq.client.producer.SendResult;
   import org.apache.rocketmq.common.message.Message;
   import org.apache.rocketmq.remoting.exception.RemotingException;
   
   import java.io.UnsupportedEncodingException;
   
   
   public class Producer {
       public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
           // 1.创建一个发送消息的对象Producer
           DefaultMQProducer producer = new DefaultMQProducer("group1");
           // 2.设定发送的命名服务器地址
           producer.setNamesrvAddr("81.68.182.114:9876");
           // 3.启动发送的服务
           producer.start();
           for (int i = 0; i < 10; i++) {
               // 4.创建要发送的消息对象, 指定topic, 指定内容body
               Message msg = new Message("topic1", ("hello rocketmq" + i).getBytes("UTF-8"));
               // 5.发送消息
               SendResult result = producer.send(msg);
               System.out.println("返回结果: " + result);
           }
           // 6.关闭连接
           producer.shutdown();
       }
   }
   ```

2. 消费者

   ```java
   package xyz.rtx3090.oneToMany;
   
   import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
   import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
   import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
   import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
   import org.apache.rocketmq.client.exception.MQClientException;
   import org.apache.rocketmq.common.message.MessageExt;
   import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;
   
   import java.util.List;
   
   public class Consumer {
       public static void main(String[] args) throws MQClientException {
           // 1.创建一个接受消息的对象Consumer
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
           // 2.设定接收的命名服务器地址
           consumer.setNamesrvAddr("81.68.182.114:9876");
           // 3.设置接受消息对应的topic, 对应的sub标签为任意
           consumer.subscribe("topic1", "*");
           // 4.设置当前消费者的消费模式(默认模式:负载均衡)
           consumer.setMessageModel(MessageModel.CLUSTERING);
           //广播模式
           //consumer.setMessageModel(MessageModel.BROADCASTING);
           // 5.开启监听, 用于接收消息
           consumer.registerMessageListener(new MessageListenerConcurrently() {
               @Override
               public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                   // 遍历消息
                   for (MessageExt msg: list) {
                       System.out.println("收到消息: " + msg);
                       System.out.println("消息是: " + new String(msg.getBody()));
                   }
                   return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
               }
           });
           // 6.启动接收消息的服务
           consumer.start();
           System.out.println("接收消息服务已经开启!");
           // 7.不要关闭消费者!
       }
   }
   ```

3. 开启"消费者Consumer"的并行运行.

   ![image-20211221195240295](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211221195240295.png)

4. 运行两次Consumer, 以此启动两个Consumer.

5. 启动Producer.

> 负载均衡模式: 多个消费者平均轮换的消费生产者发送的消息
>
> 广播模式: 多个消费者每个都完整的消费生产者发送的消息

## 多生产者多消费者

 多生产者产生的消息可以被同一个消费者消费，也可以被多个消费者消费

# 消息类别

## 概述

1. 同步消息
2. 异步消息
3. 单向消息

## 同步消息

特征: 即时性较强, 重要的消息, 且必须有回执的消息, 例如: 短信, 通知(转账成功)

**生产者 代码实现**

```java
package xyz.rtx3090.oneToMany;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;


public class Producer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 2.设定发送的命名服务器地址
        producer.setNamesrvAddr("81.68.182.114:9876");
        // 3.启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 4.创建要发送的消息对象, 指定topic, 指定内容body
            Message msg = new Message("topic1", ("hello rocketmq" + i).getBytes("UTF-8"));
            // 5.发送消息(同步发送消息)
            SendResult result = producer.send(msg);
            System.out.println("返回结果: " + result);
        }
        // 6.关闭连接
        producer.shutdown();
    }
}
```

## 异步消息

特征: 即时性较弱, 但需要有回执的消息, 例如订单中的某些信息.

**生产者 代码实现**

```java
package xyz.rtx3090.oneToMany;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.util.concurrent.TimeUnit;


public class Producer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 2.设定发送的命名服务器地址
        producer.setNamesrvAddr("81.68.182.114:9876");
        // 3.启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 4.创建要发送的消息对象, 指定topic, 指定内容body
            Message msg = new Message("topic1", ("hello rocketmq" + i).getBytes("UTF-8"));
            // 5.发送消息(异步消息)
            producer.send(msg, new SendCallback() {
                // 表示发送消息成功的返回结果
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println(sendResult);
                }
                // 表示发送消息失败的返回结果
                @Override
                public void onException(Throwable throwable) {
                    System.out.println(throwable);
                }
            });
            System.out.println("消息" + i + "发完了, 做业务逻辑去了!");
        }
        //休眠10秒
        TimeUnit.SECONDS.sleep(10);
        // 6.关闭连接
        producer.shutdown();
    }
}
```

## 单向消息

**生产者 代码实现**

特征: 不需要有回执的消息, 例如日志类消息

```java
package xyz.rtx3090.oneToMany;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.util.concurrent.TimeUnit;


public class Producer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 2.设定发送的命名服务器地址
        producer.setNamesrvAddr("81.68.182.114:9876");
        // 3.启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 4.创建要发送的消息对象, 指定topic, 指定内容body
            Message msg = new Message("topic1", ("hello rocketmq" + i).getBytes("UTF-8"));
            // 5.发送消息(单向消息)
            producer.sendOneway(msg);
            System.out.println("消息" + i + "发完了, 做业务逻辑去了!");
        }
        // 6.关闭连接
        producer.shutdown();
    }
}
```

## 延时消息

**生产者 代码实现**

消息发送时并不直接发送到服务器上, 而是根据设定的等待时间到达, 起到延时到达的缓冲作用.

```java
package xyz.rtx3090.oneToMany;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.util.concurrent.TimeUnit;


public class Producer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 2.设定发送的命名服务器地址
        producer.setNamesrvAddr("81.68.182.114:9876");
        // 3.启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 4.创建要发送的消息对象, 指定topic, 指定内容body
            Message msg = new Message("topic1", ("hello rocketmq" + i).getBytes("UTF-8"));
            // 5.发送消息(延时消息)
            // 设置延时登等级3, 这个消息将在10s之后发送(现在只支持固定的几个时间, 详情看delayTimeLevel)
            msg.setDelayTimeLevel(3);
            SendResult sendResult = producer.send(msg);
            System.out.println("返回结果: " + sendResult);
        }
        // 6.关闭连接
        producer.shutdown();
    }
}
```

> 目前支持的消息时间:
>
> private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 
>
> 20m 30m 1h 2h"; 

## 批量消息

批量发送消息能显著提供传递小消息的性能

**生产者 代码实现**

```java
package xyz.rtx3090.oneToMany;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;


public class Producer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 2.设定发送的命名服务器地址
        producer.setNamesrvAddr("81.68.182.114:9876");
        // 3.启动发送的服务
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 4.创建要发送的消息对象, 指定topic, 指定内容body
            List<Message> msgList = new ArrayList<>();
            Message msg1 = new Message("topic1", ("hello rocketmq1".getBytes("UTF-8")));
            Message msg2 = new Message("topic1", ("hello rocketmq2".getBytes("UTF-8")));
            Message msg3 = new Message("topic1", ("hello rocketmq3".getBytes("UTF-8")));
            msgList.add(msg1);
            msgList.add(msg2);
            msgList.add(msg3);
            // 5.发送消息(批量发送消息)
            SendResult result = producer.send(msgList);
            System.out.println("返回结果: " + result);
        }
        // 6.关闭连接
        producer.shutdown();
    }
}
```

> 批量发送消息 注意事项:
>
> 1. 这些批量消息应该有相同的topic
>
> 2. 相同的waitStoreMsgOK
>
> 3. 不能是延时消息
>
> 4. 消息内容总长度不超过4M
>
>    消息内容总长度包含如下:
>
>    + topic (字符串字节数)
>    + body (字节数长度)
>    + 消息追加的属性 (key与value对应字符串字节数)
>    + 日志 (固定20字节)

# 架构设计

## 技术架构

RocketMQ架构主要分为四部分, 如上图所示:

+ **Producer:** 消息发布的角色, 辞职分布式集群方式部署. Producer通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递, 投递的过程支持快速失败并且低延迟.
+ **Consumer:** 消息消费的角色, 支持分布式集群方式部署. 支持以push推, pull啦两种模式对消息进行消费. 同时也支持集群方式和广播方式的消费, 它提供实时消费订阅机制, 可以满足大多数用户的需求.
+ **NameServer:** NameServer是一个非常简单的Topic路由注册中心, 其角色类似Dubbo中的zookeeper, 支持Broker的动态注册与发现. 主要报考两个功能: Broker管理, NameServer接受Broker集群的注册信息并且保存下来作为路由信息的基本数据.