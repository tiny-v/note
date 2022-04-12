## Channel

<b>参考文档:</b>

> https://www.rabbitmq.com/channels.html (官方 channel 文档)

### 一、基础介绍

有些应用需要与 AMQP 代理建立多个连接。无论怎样，同时开启多个TCP连接都是不合适的，因为这样做会消耗掉过多的系统资源。AMQP 0-9-1 提供了通道（channels）来处理多连接，可以把channel理解成共享一个 TCP 连接的多个轻量化连接（通常每个thread创建单独的channel进行通讯）

客户端执行的每个协议操作都发生在Channel上。Channel上的通信与另一个Channel上的通信完全分开，因此每个协议方法还带有一个Channel ID，Broker和Client都使用一个整数来确定Channel。

Channel仅存在于Connection的上下文中，从不单独存在。当Connection关闭时，其上的所有Channel也会关闭。

### 二、资源消耗

每个Channel在客户端上消耗的内存相对较少, 具体取决于客户端库的实现细节。 它还可以使用一个专用的Channel池（类似的线程池)，在执行具体操作， 从channel池中获取channel。

每个Channel还消耗客户端连接的节点上相对少量的内存，以及一些 Erlang 进程。由于一个节点通常服务于多个Channel连接，因此过多的Channel使用或Channel泄漏的影响将主要反映在 RabbitMQ 节点的指标上，而不是客户端的指标上。

鉴于这两个因素，强烈建议限制每个连接使用的Channel数。根据经验，大多数应用程序可以在每个连接中使用一位数的Channel数。那些并发率特别高的（通常这样的应用程序是消费者）可以从每个线程/进程/协程一个Channel开始，当指标表明原始模型不再可持续时切换到Channel池，例如。因为它消耗了太多的内存。

### 三、连接池（Connection & Channel）