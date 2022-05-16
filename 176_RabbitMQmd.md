## Hello World

P (producer / publisher) : 生产者，发送消息的服务

C（consumer）：消费者，接受消息的服务

红色区域就是MQ中的Queue，可以把它理解成一个邮箱

+ 首先信件来了不强求必须马上去拿
+ 其次，它是有最大容量的（受不急和磁盘的限制，是一个缓存区）
+ 允许多个消费者监听同一个队列，争抢消息

## Worker模型

Worker模型中也只有一个工作队列，但是它是一种竞争消息模式。可以看到同一个队列我们绑定了多个消费者，消费者中争抢着消费信息，这可以有效的避免消息堆积。

比如对于短信微服务集群来说可以用这种消息模型，来了请求大家抢着消费掉。

如何实现这种架构：对于上面的HelloWorld其实就是相同的服务我们启动了多次罢了，自然就是这种架构了。

## 订阅模型

订阅模型借助一个新的概念：Exchange（交换机）实现，不同的订阅模型本质上是根据交换机（Exchange）类型划分的。

订阅模型有三种

1. Fanout（广播模型）：将消息发送给绑定给交换机的所有队列（因为他们使用的是同一个RoutingKye）
2. Direct（定向）：把消息发送给拥有指定Routing Key（路由键）的队列
3. Topic（通配符）：把消息传递给拥有符合Routinh Patten（路由模式）的队列

## 订阅之Fanout模型

这个模型的特点就是它在发送消息的时候，并没有指明Rounting Key，或者说他指定了Ruting Key，但是所以的消费者都知道，大家都能接受到消息，就像听广播。

## 订阅之Direct模型

P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接受生产者的消息，然后把消息递交给与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key为error的消息

C2：消费者，其所在队列指定了需要routing key为info、error、warning的消息

拥有不同的RoutingKey的消费者，会收到来自交换机的不同信息，而不是大家都使用同一个Routing key和广播模型区分开来。

## 订阅之Topic模型

类似于Direct模型，区别是Topic的Routing key支持通配符



