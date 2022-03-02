## 死信队列(DLX)

> 死信队列, DLX全称为Dead-Letter-Exchange , 可以称之为死信交换机，也有人称之为死信邮箱。 当消息在一个队列中变成死信(dead message)之后，它能被重新发送到另一个交换机中，这个交换机就是DLX ，绑定DLX的队列就称之为死信队列.

### 什么样的消息会进入死信队列
> 1. 消息被拒绝（basic.reject/ basic.nack）, 并且不再重新投递 requeue=false
> 
> 2. TTL(time-to-live) 消息超时未消费
> 
> 3. 达到最大队列长度


