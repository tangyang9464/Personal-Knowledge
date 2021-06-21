## Netty

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
- **IO多路复用**：通过系统调用select\epoll来查询IO文件描述符的就绪状态，从而对就绪的描述符发起read\write系统调用。达到一个线程管理多个连接的效果
- **AIO**：向内核注册IO操作，整个IO过程交给内核处理，完成后通知用户进行后续处理

### Java NIO

- **Buffer**：本质上是一个数组，拥有capacity（容量）、limit（读写上限）、position（读写的当前位置）、mark（临时保存position）

- **Channel**：一个Socket连接用一个Channel来表示。更加可以用一个Channel来表示一个文件描述符
- **Selector**：选择器用于管理多个连接，也就是多个Channel
	- **SelectionKey**：表示需要监控的事件类型，包括可读、可写、连接、接收；一一对应Channel

### Reactor模式（反应器模式）

具有reactor和handler两个组件，reactor负责轮询监听IO事件，在就绪时将任务分发给相应的handler进行非阻塞的处理
