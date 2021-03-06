# 基础篇


### 主要概念及重要参数

* producer：生产者，投递消息的一方
* consumer：消费者，接收消息的一方
    * prefetch ：预取默认值
* broker:消息中间件的服务节点
* queue:队列
    * rabbitmq的内部对象，用于存储消息。rabbitmq中的消息只能存储在队列中。
    * 多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊（Round-Robin,及轮询）给多个消费者，而不是每个消费者都收到所有的消息。
    * rabbitmq不支持队列层面上的广播消费。
* exchange:生产者将消息发送到交换器，由交换器将消息路由到一个或多个队列中。
    * direct:此类型交换器会将消息投递到，routingKey和bindingKey完全相同的队列中。
    * fanout:它会把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。
    * topic:routingKey和bindingKey会按照一定的规则匹配,消息会被发送到匹配中的队列上去。规则如下
    bindingKey中可以存在两种特殊字符*和#,*用于匹配一个单词，#用于匹配多个单词。
    ![](../../../pic/中间件/rabbitmq_2.png)
    * headers:不根据路由键匹配规则来路由消息，而是根据发送消息内容中的headers属性进行匹配。此类型交换器性能较差，使用较少。
    
* routingKey:路由键，生产者将消息发送给交换器，一般会指定一个routingKey,用于指定消息的路由规则。
* bindingKey:用于将交换器和队列绑定。
* connection:消费者和生产者需要与broker建立TCP连接，也就是connection
* channel:AMQP信道，每个信道都会被指派一个唯一的ID,信道是建立在connection之上的虚拟连接，每一条AMQP指令都是通过信道完成的。TCP连接的建立和销毁是非常昂贵的开销，信道相当于轻量级连接，
* vhost:
    * virtual host只是起到一个命名空间的作用，所以可以多个user共同使用一个virtual host，文章开头写的vritual_host = '/'，这个是系统默认的。
    * vhost是rabbitmq分配权限的最小细粒度,比如我们可以为一个用户分配一个可以访问哪个或者哪一些vhost的权限,但是不能为用户分配一个可以访问哪一些exchange，或者queue的权限，因为rabbitmq的权限细粒度没有细化到交换器和队列，他的最小细粒度是vhost(vhost中包含许多的exchanges，queues，bingdings).所以如果exchangeA 和queueA 只能让用户A访问，exchangeB 和queueB 只能让用户B访问，要达到这种需求，只能为exchangeA 和queueA创建一个vhostA，为exchangeB 和queueB 创建vhostB，这样就隔离开来了。
    * 一个broker可以开设多个vhost，用于不同用户的权限分离.











