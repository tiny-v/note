# Connection 和 Channel 的关系


### 参考文档
> https://www.rabbitmq.com/connections.html （官方 connection 文档）

## Connection

> 应用程序使用客户端库与 RabbitMQ 交互, RabbitMQ 支持的所有协议都是基于 TCP 的，并默认使用长期连接，且一个客户端库连接使用单个 TCP 连接。

> 当不再需要连接时，应用程序必须关闭它们以节省资源。未能做到这一点的应用程序将面临最终耗尽其目标资源节点的风险。

> 操作系统对单个进程可以同时打开的 TCP 连接数量有限制。该限制对于开发和一些 QA 环境通常是足够的。生产环境必须配置为使用更高的限制，以支持更大数量的并发客户端连接。

