# 进阶篇

### 1.发送端确认

#### 事务机制
channel.txSelect,channel.txCommit,channel.txRollback用于将信道设置为事务模式，
```
channel.txSelect();
channel.basicPublish(EXCHANGE  NAME , ROUTING  KEY ,MessageProperties . PERSISTENTTEXT_PLAIN ,"transaction  messages".getBytes());
channel.txCommit();
```
* 事务模式的四个步骤：
    * 1.tx.select，将信道置为事务模式
    * 2.broker回复tx.select-ok,确认已经将信道置为事务模式
    * 3.在消息发送完成后，broker发送tx.commit提交事务
    * 4.broker回复tx.commit-ok，确认事务提交 
* 缺点：事务机制在发送一条消息之后会使发送端阻塞，会降低mq的吞吐量

#### 发送方确认（comfirm模式）
生产者将信道置为confirm模式后，所有在该信道上的消息后对被指派一个确认的Id,一旦消息被投递到匹配的队列之后，rabbitmq就会异步发送一个确认给生产者（包含消息的唯一Id）,这就使得生产者知晓消息已经达到目的地了。如果消息和队列设置了持久化，确认消息会在写入磁盘后发出。
>下面代码为在springboot中的用法
```
@Component
public class MsgDirectSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;


    public boolean send(String msg, String orderNo) {
        rabbitTemplate.setExchange(ExchangeConstant.DIRECT_EXCHANGE_A);
        rabbitTemplate.setRoutingKey(RouteKeyConstant.ROUTEKEY_A);
        rabbitTemplate.setConfirmCallback((correlationData, ack, s) -> {
            String sourceOrderNo = correlationData.getId();
            if (ack) {
                System.out.println("订单:"+sourceOrderNo+"推送成功");
            } else {
                System.out.println("订单:"+sourceOrderNo+"推送失败");
            }
        });
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            System.out.println("================");
            System.out.println("message = " + message);
            System.out.println("replyCode = " + replyCode);
            System.out.println("replyText = " + replyText);
            System.out.println("exchange = " + exchange);
            System.out.println("routingKey = " + routingKey);
            System.out.println("================");
        });
        CorrelationData correlationData=new CorrelationData(orderNo);
        rabbitTemplate.correlationConvertAndSend(msg, correlationData);
        return true;
    }
}
```
* comfirmCallBack:始终会回调，根据ack参数判断是否成功到达exchange。
* returnCallBack:表示如果你的消息已经正确到达交换机，但是没有到达queue，那么就会回调returnCallBack，并且把信息送回给你。（必须 rabbitTemplate.setMandatory(true)，不然当发送到交换器成功，但是没有匹配的队列，不会触发 ReturnCallback 回调。而且 ReturnCallback 比 ConfirmCallback 先回调）。
* 不能单单依靠 ConfirmCallback 的 ack 返回值为 true，就断定当前消息发送成功了,有可能并没有到达队列，却这种情况下，会先回调returnCallBack，然后confirmcallBack的ack为true.


### 2.消费端确认（ACK）
消费者在订阅队列时，可以指定autoAck参数
* 自动确认（autoAck=true）：rabbitmq会自动把发送出去的消息置为确认，然后从内存(或磁盘)中删除，而不管消费者是否真正地消费到了这些消息。
* 手动确认（autoAck=false）:此时队列中会从在两种消息，一部分是等待投递给消费者的消息，一部分是已经投递给消费者，但是没有收到消费者确认信号的消息。如果rabbitmq一直没有收到消费者确认，且消费者已经断开,则消息会重新入队。
>下面代码为在springboot中的用法
```
 @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = QueueConstant.QUEUE_E, durable = "true"),
            exchange = @Exchange(value = ExchangeConstant.FANOUT_EXCHANGE_A)))
    public void receiveFanout(Message message, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws Exception {
        try {
            String msg = new String(message.getBody());
            System.out.println("收到消息：" + msg);
            
            //执行业务逻辑
            
            //返回ack
            channel.basicAck(tag, false);
        } catch (Exception e) {
            // true: 如果被拒绝的消息应该重新排队，否则为false
            channel.basicReject(tag, true);
        }
    }
```
 * basicReject(long deliveryTag, boolean requeue)：deliveryTag：可以看做消息的编号。requeue=true用于表示消息将重新入队，requeue=false表示消息将被从队列中移除。
 * basicAck(long deliveryTag, boolean multiple) :multiple=false:表示单条模式,multiple=false:表示多条模式，即确认deliveryTag编号之前的所有未被确认的消息。
 * basicNack(long deliveryTag, boolean multiple, boolean requeue) ：multiple=false时和basicReject相同的作用。
 * Basic.RecoverOk  basicRecover(boolean  requeue):TODO

### 2.死信队列

#### what
* 死信：消息被拒绝（basic.reject/basic.Nack），且requeue=false即不在重新入队；消息过期；队列达到最大长度


#### why
死信队列实际上就是，当我们的业务队列处理失败(比如抛异常并且达到了retry的上限)，就会将消息重新投递到另一个Exchange(Dead Letter Exchanges)，该Exchange再根据routingKey重定向到另一个队列，在这个队列重新处理该消息




### 3.延迟队列

### 4.优先级队列


#### 参考文章
[Introducing Publisher Confirms](https://www.rabbitmq.com/blog/2011/02/10/introducing-publisher-confirms/)