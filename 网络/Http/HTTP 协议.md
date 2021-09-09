# HTTP 
参考文档:
> 1. https://cloud.tencent.com/developer/article/1464938 (详解HTTP/1.0、HTTP/1.1、HTTP/2、HTTPS)
> 2. https://baike.baidu.com/item/http/243074 (百度百科)
> 3. https://zhuanlan.zhihu.com/p/86426969 (TCP连接三次握手四次挥手)

## OSI七层模型简介
![OSI七层模型](./images/OSI七层模型.png)


## 发展史
> 1. HTTP/0.9：1991年发布，极其简单，只有一个get命令
> 2. HTTP/1.0：1996年5月发布，增加了大量内容
> 3. HTTP/1.1：1997年1月发布，进一步完善HTTP协议，是目前最流行的版本
> 4. SPDY ：2009年谷歌发布SPDY协议，主要解决HTTP/1.1效率不高的问题
> 5. HTTP/2 ：2015年借鉴SPDY的HTTP/2发布

###  HTTP/0.9

  HTTP协议的最初版本，功能简陋，仅支持请求方式GET。 是一个交换信息的无序协议，仅仅限于文字。由于无法进行内容的协商，在双发的握手和协议中，并有规定双发的内容是什么，也就是图片是无法显示和处理的。

###  HTTP/1.0

#### 1 增加内容

 *  请求和响应支持Header，用来描述一些元数据.
 *  请求方式除了GET外，增加了请求方式POST和HEAD
 *  响应的数据格式，不仅限于HTML, 根据 **Content-Type** 支持多种格式， 比如 text/html、image/jpeg等
 *  状态码(status code)
 *  多字符集支持 
 *  多部分发送(multi-part type)
 *  权限(authorization)
 *  缓存(cache) 
 *  内容编码(content encoding）

   &nbsp;&nbsp;&nbsp;。。。。

#### 2 缺陷 
客户端和服务端只保持短暂的连接，客户端每次请求都需要与服务端建立一个TCP连接。（TCP连接的新建成本很高，因为需要客户端和服务端三次握手），
服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求

###  HTTP/1.1
#### 1 增加内容：

 * 长连接(Persistent Connection)：</br>
   允许HTTP设备在事务处理结束之后将TCP连接保持在打开的状态，以便未来的HTTP请求重用现在的连接，直到客户端或服务器端决定将其关闭为止。
   在HTTP1.0中使用长连接需要添加请求头 Connection: Keep-Alive，而在HTTP 1.1 所有的连接默认都是长连接，除非特殊声明不支持(HTTP请求报文首部加上Connection: close)
   
   &nbsp;
* chunked编码传输：</br>
   该编码将实体分块传送并逐块标明长度，直到长度为0块表示传输结束，这在实体长度未知时特别有用(比如由数据库动态产生的数据);
 

* 字节范围请求：</br>
  HTTP1.1支持传送内容的一部分。比方说，当客户端已经有内容的一部分，为了节省带宽，可以只向服务器请求一部分。该功能通过在请求消息中引入了range头域来实现，它允许只请求资源的某个部分。
  在响应消息中Content-Range头域声明了返回的这部分对象的偏移值和长度。如果服务器相应地返回了对象所请求范围的内容，则响应码206（Partial Content）
 

* Pipeline: 即在同一个TCP连接中，客户端可以同时发送多个请求
 

* 请求消息和响应消息都支持Host头域：</br>
  在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。
  但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。


* 缓存处理：</br>
  HTTP/1.1在1.0的基础上加入了一些cache的新特性，引入了实体标签，一般被称为e-tags，新增更为强大的Cache-Control头。
  

* 新增请求方法：</br>
  HTTP1.1增加了OPTIONS, PUT, DELETE, TRACE, CONNECT方法;
  

#### 2. 缺点
* 队头阻塞(Head-of-line blocking)：
HTTP/1.1 的持久连接和管道机制允许复用TCP连接，在一个TCP连接中，也可以同时发送多个请求，但是所有的数据通信都是按次序完成的，服务器只有处理完一个回应，才会处理下一个回应。
比如客户端需要A、B两个资源，管道机制允许浏览器同时发出A请求和B请求，但服务器还是按照顺序，先回应A请求，完成后再回应B请求，这样如果前面的回应特别慢，后面就会有很多请求排队等着，这称为"队头阻塞(Head-of-line blocking)"

### SPDY 
2009年谷歌发布SPDY协议，主要解决HTTP/1.1效率不高的问题

### HTTP/2.0

####  HTTP/2.0 主要是对上个版本的性能优化

* 二进制分帧（Binary Format）- http2.0的基石

* 多路复用 (Multiplexing) / 连接共享

* 头部压缩（Header Compression）

* 请求优先级（Request Priorities）

* 服务端推送（Server Push）



#### 2 HTTP/2.0 性能优化


## HTTPS




