# IM即时通讯系统

## 项目流程

1.   TCP连接。客户端通过zookeeper获取可用服务器，使用负载均衡算法选择服务器进行连接。连接成功后将userid和serverInfo保存在redis
2.   客户端发送握手包，包括userid。
3.   服务端进行登录验证，主要验证是否重复登录。验证成功发送响应包，并保存相应的userid和channel上下文
4.   客户端收到响应包后启动心跳计时器。使用NioEventLoop的定时器
5.   开始通信

## 项目功能

### 私聊

通过命令显示当前在线用户(redis)，客户端给对应用户发消息，服务端接收到消息后，判断touser是否在当前服务器(redis)，若在直接转发。若不在，就发到kafka相应用户的topic。另一服务器订阅topic消费，找到相应的用户转发消息。

### 群聊

服务器接收消息后发到group的topic，所有服务器都订阅该topic，然后消费消息转发给自己服务器的所有用户。

## 网路连接可靠性

### 心跳检测

客户端握手成功就会启动定时器发送心跳包。通过userTrigger来判断心跳超时。若N次没有得到响应，那么就尝试重新连接或者断开连接

### 断线重连

- 启动失败重连
	通过回调函数进行重连
- 运行服务器掉线
	通过handler的Inactive方法监听掉线，如果掉线就重新选择服务器进行连接

### 消息即时性

维持长连接

### 消息可靠性

TCP连接，Kafka发送重试以及持久化机制

### 消息安全性

http的话可以直接使用https
通过非对称加密

## ProtoBuf

-    序列化框架的评价标准
     1. 编码后的码流长度
     2. 编解码的性能
     3. 是否支持跨语言，支持的语言种类
     4. API使用简便性
-    优势
  -    性能好
  -    文本化的数据结构描述语言，易兼容和维护，可以实现多语言，平台无关。

## 多种方案

1. HTTP接口调用

	设置一个路由层，发送消息通过http接口，通过路由层实现群发或者私聊，路由层也是通过调用服务器http接口。然后服务器回复消息使用netty

2. 服务器之间长连接

	服务器与其他服务器进行通信转发，相当于自己进行中转。维护的连接比较多，每个服务器既是netty客户端又是服务端，比较麻烦

3. 消息队列中转

	逻辑清晰，模块耦合度低，易于维护

## WebSocket

实现全双工通信，改变了HTTP服务器不能主动push消息的弊端。

-    轮询：客户端不停发送请求，服务端立即响应，达到一种push消息的功能
-    长轮询：服务器只当有新数据的时候才响应，否则就挂起请求。

## 为什么不使用Tomcat

tomcat是基于http协议的。而netty既是基于NIO的多路复用机制，又能自定义通信格式

## 自定义通信格式

|  字段  | 长度 |                描述                 |
| :----: | :--: | :---------------------------------: |
|  魔数  |  1   |                0xAB                 |
|  长度  |  4   |           除魔数外的长度            |
| 包类型 |  1   | 包类型，包括握手、心跳、群聊-私聊包 |
| 发送者 |  4   |              发送者id               |
| 接收者 |  4   |              接收者id               |
| 数据包 | 不定 |           发送的消息主体            |

魔数4+包类型1+发送者4+接收者4+数据包
包类型包括握手包，心跳包，群聊包，私聊包
握手包用于确认客户端是否重复登录

## Netty

### 模型

- BossGroup：负责接收连接，将任务交给Work处理
- WorkGroup：负责实际的任务处理

###  粘包拆包

- **原因**：底层网络通过TCP内核缓冲区读写。当缓冲区数据包较大，就可能会将一个包分成多个ByteBuf，用户读到半包

  ​	较小时，ByteBuf读到的可能就是多个包，导致粘包。

- **解决**

  - 基于固定长度：FixedLengthFrameDecoder
  - 基于分隔符：LineBasedFrameDeCoder-行分隔符。DelimiterBasedFrameDeCoder-任意分隔符。
  - 基于任意长度：LengthFieldBasedFrameDeCoder，可在数据包之前添加长度字段，用于解决粘包问题
    - 四个参数：数据包最大长度、长度字段起始位置（偏移量）、长度和数据之间夹杂的字段长度（如版本号）、数据包偏移量

### 高并发IO底层原理

#### IO读写的基本原理

- 用户程序的IO读写依赖于两个系统调用read，write。它们的功能只是将数据从内核缓冲区复制到程序缓冲区
- 与物理设备的读写完全由操作系统内核完成，用户无感知

#### 四种IO模型

- **BIO**：用户主动发起IO请求，阻塞等待内核IO完成后才返回用户空间

- **NIO**：用户主动发起IO请求，不需要等待内核IO彻底完成便可立即返回用户空间

  客户端A发送请求给请求线程了，请求线程把请求交给工作线程处理了。**请求线程将任务分派给别的任务之后就空闲了**，此时请求线程又可以处理别的请求了（不管前一个工作线程是否处理完）。

  比如客户端A提交了一次IO请求，可能此次IO请求在上述所说的工作线程没处理完，但是 客户端A还能发出IO请求，而服务器同样能接收这个请求（因为请求线程是空闲的）。此时，客户端请求服务器来说，我们就认为是非阻塞的。

- **IO多路复用**：通过系统调用select\epoll来查询IO文件描述符的就绪状态，从而对就绪的描述符发起read\write系统调用。达到一个线程管理多个连接的效果

- **AIO**：向内核注册IO操作，整个IO过程交给内核处理，完成后通知用户进行后续处理

### Java NIO

- **Buffer**：本质上是一个数组，拥有capacity（容量）、limit（读写上限）、position（读写的当前位置）、mark（临时保存position）

- **Channel**：一个Socket连接用一个Channel来表示。更加可以用一个Channel来表示一个文件描述符
- **Selector**：选择器用于管理多个连接，也就是多个Channel
  - **SelectionKey**：表示需要监控的事件类型，包括可读、可写、连接、接收；一一对应Channel

### ByteBuf

本质是一个数组，可动态扩展，维护了读写两个指针，用于支持顺序读写的操作。

- **堆内存**：后端编解码模块使用

- **直接内存**：IO网络读写缓冲区使用，可以免去一次复制

  即堆外内存，直接向系统申请的内存。
  它读写socketChannel会少一次拷贝
  正常流程：内核socket--用户--堆外内存--堆内存

- ``byteBuf.alloc()``会得到一个``ByteBufAllocator``，用于分配内存

  其中``unpool``是直接申请一块新内存，``pool``是从已申请的内存中拿一块

### 编解码器编程细节

- **decode**-**ByteToMessage**：

  ```Java
  // 1.开拓一块ByteBuf用于接收部分字节
  // 相当于ByteBufAllocator.heapBuffer，使用的是堆内存
  ByteBuf byteBuf = Unpooled.buffer(len);
  in.readBytes(byteBuf);
  try {
      // 2.将bytebuf转换成字节数组
      byte[] bytes = byteBuf.array();
      // 3.将字节数组通过protobuf解码成msg对象
      MsgProtobuf.ProtocolMsg msg = MsgProtobuf.ProtocolMsg.parseFrom(bytes);
      if (msg!=null){
          out.add(msg);
      }
  } catch (InvalidProtocolBufferException e) {
      e.printStackTrace();
  }
  ```

- **encode**-**MessageToByte**：

  ```Java
  // protobuf编码成字节数组，然后bytebuf写入就好了
  byte[] bytes = protocolMsg.toByteArray();
  byteBuf.writeInt(bytes.length);
  byteBuf.writeBytes(bytes);
  ```

### Reactor模式（反应器模式）

具有reactor和handler两个组件，reactor负责轮询监听IO事件，在就绪时将任务分发给相应的handler进行非阻塞的处理

# RPC系统

## 使用流程

客户端使用动态代理生成代理类，通过代理类调用方法。

调用方法会从zookeeper中获取服务地址，通过负载均衡算法挑选出可用服务地址。使用netty进行TCP连接。发送接口名，方法名，方法类型，方法参数。

服务端收到消息后，通过map查找出相应的实现类，通过反射调用方法得到结果再返回。

## 服务治理

zookeeper保存serviceName--serverInfo（ip-port）

客户端通过服务名称去寻找可用的服务器

服务端启动时候注册服务，并将serviceName--Object（实现类）保存在本地map，方便通过客户端发过来的接口名得到实现类，从而通过反射执行方法

## 消息格式

数据包长度--接口名--方法名--方法类型--方法参数

## 与Http区别

1. HTTP 指的是通信协议，而 RPC 则是远程调用，它基于TCP/IP协议。可以直接使用http协议，也可以自定义协议
2. RPC要求服务提供方和服务调用方都需要使用相同的技术，比如要么都hessian，要么都dubbo
   而http无需关注语言的实现，只需要遵循相应规范

# 实习

## 全局异常处理

通过SpringBoot的`ControllerAdvice`注解创建全局异常处理类，通过`ExceptionHandler`注解指定不同异常的处理类

统一返回结果对象`ResponseResult`，包括code、msg、data。

大体种类有：

1000--登录错误

1001--VIP权限错误

2000--数据库token错误

非微信平台打开错误

## 拦截器实现登录验证

实现`HandlerInterceptor`接口，并在实现`WebMvcConfigurer`接口并注册拦截器

以前的方式是在指定的controller里面进行重复编码验证

## 策略模式替换if/else

需求是后台微信菜单需要根据不同指令做不同的事情。比如点击事件，用户点击按钮后台返回联系方式。订阅事件，用户订阅后后台发送欢迎语。

因为有很多种事件，用了多重if/else，不是很好维护。

定义一个接口，有一个process方法。不同事件实现同一个接口。有一个context类根据事件名称通过反射获取到对应的实现类。其中事件名称和对应实现类的限定名用一个枚举类来维护。

