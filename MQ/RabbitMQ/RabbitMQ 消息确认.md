## 消息确认

### 参考文档
> https://juejin.cn/post/6997558113242972191 (必知必会！RabbitMQ消息确认机制)

### 一、消息发送确认
> 该过程分成两步：
> 
> 1.确认生产者是否成功将消息发送到Broker
> 
> 2.Broker上的交换机再将消息投递给队列 queue的过程

```
Tips: 使用两个消息回调之前需要到application.yml中配置一下消息确认回调、消息失败回调。

1. publisher-confirm-type: 表示确认消息的类型，有(none | correlated | simple)这三种类型。

none：表示禁用发布确认模式，默认值，使用此模式之后，不管消息有没有发送到Broker都不会触发ConfirmCallback回调。
correlated：表示消息成功到达Broker后触发ConfirmCalllBack回调
simple：simple模式下如果消息成功到达Broker后一样会触发ConfirmCalllBack回调，发布消息成功后使用rabbitTemplate调用waitForConfirms或waitForConfirmsOrDie方法等待broker节点返回发送结果，根据返回结果来判定下一步的逻辑，🚨注意：waitForConfirmsOrDie方法如果返回false则会关闭channel信道，则接下来无法发送消息到broker。


2. publisher-returns: true表示开启失败回调，开启后当消息无法路由到指定队列时会触发ReturnCallback回调。

```

#### 1.1 生产者到Broker的确认 (ConfirmCallback)

```
发送消息时会触发该回调，消息投递到Broker成功或者失败都会返回一个ConfirmCallback，我们可以通过返回的ack来判断消息是否达到Broker。

ConfirmCalllBack是一个接口，我们需要实现该接口并重写其comfirm方法。

@Override
public void confirm(CorrelationData correlationData, boolean ack, String s) {
    //
}

correlationData: 是用来做消息的唯一标识，我们在发送消息时可以附带这个唯一标识，在该回调中可以获取到当前消息的唯一标识。
ack: 表示消息发送到Broker的状态，true表示送达，false反之。
s: 表示发送失败的原因。
```
#### 1.2 Exchange到Queue的确认 (ReturnCallback)

```
如果消息未能路由到目标队列则将触发回调ReturnCallback，ReturnCallback也是一个接口，我们需要实现该接口并重写其returnedMessag方法。

@Override
public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
    //
}

message: 表示的是消息体
replyCode: 表示响应码
replyText: 表示响应内容
exchange: 表示发送消息时指定的交换机
routingKey: 表示发送消息时指定的routing key

这些参数就是记录下当前消息的详细投递数据，方便我们后面根据业务的需求做消息的重发或者补偿等操作。
```


### 二、消息接收确认(ACK)

> 消费者收到消息后需要对 RabbitMQ Server 进行消息ACK确认，RabbitMQ根据确认信息决定是删除队列中的该信息还是重新发送。

#### 2.1 消息确认的三种模式

> listener.simple.acknowledge-mode: none(默认) | manual | auto

##### 2.1.1 手动确认（manual）

```
在该模式下，消费者消费消息后需要根据消费情况给Broker返回一个回执，是确认ack使Broker删除该条已消费的消息，还是失败确认返回nack，还是拒绝该消息。
开启手动确认后，如果消费者接收到消息后还没有返回ack就宕机了，这种情况下消息也不会丢失，只有RabbitMQ接收到返回ack后，消息才会从队列中被删除。
```
###### 该模式下有三种确认方式:

<1>. channel.basicAck(long deliveryTag, boolean multiple)

```
basicAck方法表示成功确认，使用此方法后，消息会被rabbitmq broker删除.

参数long deliveryTag为消息的唯一序号，
参数boolean multiple表示是否一次消费多条消息 (false:表示只确认该序列号对应的消息 | true:表示确认该序列号对应的消息以及比该序列号小的所有消息)

eg: 比如我先发送2条消息，他们的序列号分别为2,3，并且他们都没有被确认，还留在队列中，那么如果当前消息序列号为4，那么当multiple为true，则序列号为2、3的消息也会被一同确认。

```

<2>. channel.basicNack(long deliveryTag, boolean multiple, boolean requeue)

```
basicNack方法表示失败确认，一般当我们消费消息时出现异常用到此方法.

参数requeue表示是否将消息重新投递到队列. (false: 表示消息不重回队列并且丢弃该消息 | true: 表示消息重回队列)

```

<3>. basicReject(long deliveryTag, boolean requeue)

```
basicReject方法表示拒绝消息.

参数requeue表示是否将消息重新投递到队列. (false: 表示消息不重回队列并且丢弃该消息 | true: 表示消息重回队列)

该方法与basicNack方法的区别就是不支持multiple批量确认。
```

##### 2.1.2 自动确认 (none)

```
rabbitmq默认消费者正确处理所有请求。(不设置时的默认方式)
```

##### 2.1.3 根据情况确认 (auto)

```
如果消费者在消费的过程中没有抛出异常，则自动确认。

当消费者消费的过程中抛出"AmqpRejectAndDontRequeueException"异常的时候，则消息会被拒绝，且该消息不会重回队列。

当抛出"ImmediateAcknowledgeAmqpException"异常，消息会被确认。

如果抛出其他的异常，则消息会被拒绝，但是与前两个不同的是，该消息会重回队列，如果此时只有一个消费者监听该队列，那么该消息重回队列后又会推送给该消费者，会造成死循环的情况。
```