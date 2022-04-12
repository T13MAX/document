## Netty in Action



## 第一部分 Netty的概念及体系结构 (1-9)

### 第一章 Netty----异步和事件驱动

Netty是一款异步的事件驱动的网络应用程序框架, 支持快速开发可维护的高性能的面向协议的服务器和客户端

ServerSocket::accept()会一直阻塞

返回的Socket::getInputStream()/getOutputStream() 来读写 "Done"是读完 需要为每一个客户端Socket创建一个新的Thread

每个线程调用栈都要分配内存 而且可能还都处于休眠状态 即使物理上支持 上下文切换带来的开销也很大

NIO有选择器, 一个线程处理多个连接 没有IO操作的时候线程也可以干别的 但是高负载下高效准确安全的玩还得看Netty

直接使用底层的API暴露了复杂性, 用较简单的抽象隐藏底层实现的复杂性

Netty特性:

1. 设计:统一简单的API, 支持多种传输类型, 简单强大的线性模型
2. 比java的核心API更高的吞吐量以及更低的延迟, 得益于池的优化和复用, 拥有更低的资源消耗, 最少的内存复制
3. 不会因为慢速快速或超载的连接而导致内存溢出, 消除不公平读写比率
4. 完整的SSL/TLS以及StartTLS支持

异步的事件驱动的:可以以任意顺序响应在任意时间点产生的事件   

Channel: 一个到实体的开放连接 如读操作和写操作 可以看做是入站或者出站的数据载体

回调: 一个指向已经被提供给另外一个方法的方法的引用 (比如channelActive()内些)

Future: 操作完成时通知应用程序  jdk预制的Future需要手动检查对应的操作是否完成 ChannelFuture可以注册Listener 重写operationComplete方法

事件和ChannelHandler: 基于发生的事件触发适当的动作

### 第二章 第一款Netty程序

服务器:

ChannelInboundHandlerAdapter

1. channelRead() 每个传入的消息都调用
2. channelReadComplete() 读取数据完毕 ctx.writeAndFlush()冲刷到远程节点 并关闭 addListener(ChannelHandlerContext.CLOSE)
3. exceptionCaught() 读取期间 有异常就会调用 异常会顺着ChannelPipeline往下走 所以最少要有一个带异常处理的ChannelHandler

客户端:

SimpleChannelInboundHandler channelRead0() 自动释放

write是异步的 可能channelRead()返回后仍然没有完成 S负责释放ByteBuf的内存 C要ctx冲刷释放

### 第三章 Netty的组件和设计

Channel---Socket  基本的IO操作 bind() connect() read() write() 

​	NioSocketChannel NioSctpChannel NioDatagramChannel LocalServerChannel EmbeddedChannel

EventLoop---控制流 多线程处理 并发

​	一个EventLoopGroup包含一个或多个EventLoop 一个EventLoop只和一个Thread绑定 IO操作在他专有的Thread上被处理 一个Channel只注册于一个EventLoop 一个EventLoop可被分给多个Channel

ChannelFuture---异步通知	Netty的IO操作都是异步的 ChannelFuture看做是将来要执行的结果的占位符 addListener ChannelFutureListener 某操作完成时

ChannelHandler	充当了所有处理入站和出站数据的应用程序逻辑的容器 ChannelHandlerAdapter ChannelInboundHandlerAdapter ChannelDuplexHandler

ChannelPipeline	为ChannelHandler链提供了容器

编码器解码器 实现了ChannelInboundHandlerAdapter ChannelOutboundHandlerAdapter channelRead被重写 MessageToMessageDecoder/Byte Encoder

### 第四章 传输

OIO---阻塞传输

NIO---异步传输

Local---JVM内部的异步通信 

Embedded

零拷贝 使用NIO和Epoll传输时才可使用的特性 高效地将数据从文件系统移动到网络接口 无需内核用户复制 只能传输文件的原始内容

### 第五章 ByteBuf

可以被用户扩展	通过内置的复合缓冲区类型实现了透明的零拷贝	容量按需增长	在读和写两种模式之间切换不需要flip()	读写使用不同索引	支持链式调用 支持引用计数	支持池化

ByteBuf的使用模式

1. 堆缓冲区 hasArray()==true 数据存在JVM堆中 支撑数组 没有池化的情况下快速释放 非常适合于有遗留的数据需要处理的情况
2. 直接缓冲区 hasArray()==false 分配释放比较昂贵
3. 复合缓冲区 CompositeByteBuf 为多个ByteBuf提供一个聚合视图

readIndex writeIndex capacity discardReadBytes()丢弃并回收空间 clear()两个索引置0 内容不清除

indexOf() process() 查找

派生缓冲区 duplicate() slice() Unpooled.unmodifiableBuffer() order() readSlice() 返回一个新的ByteBuf 内部存储是共享的 想要真实副本使用copy()

读写操作 get set 从给定索引开始 并且索引不变 read write会调整索引 全都是xxx数据类型 比如 readInt

isReadable() readableBytes() capacity() 再次扩容直到maxCapacity hasArray()是否一个数组支撑 array()返回数组

ByteBufHolder接口 除了实际数据 还需要存储各种属性值 比如HTTP响应除了字节内容还需要状态码cookie之类的

ByteBuf分配 通过ByteBufAllocator实现池化 可以分配描述过的任意类型的ByteBuf实例 

​	PooledByteBufAllocator 最大限度减少内存碎片  默认

​	Unpooled工具类 buffer() 返回一个未池化的堆内存的 directBuffer返回一个直接内存的 wrappedBuffer() 返回一个包装了给定数据的 copiedBuffer() 复制

ByteBufUtil类 hexdump() 十六进制打印内容

引用计数 通过在某个对象所持有的资源不在被其他对象引用时释放该对象所持有的资源来优化内存使用和性能的技术

### 第六章 ChannelHandler和Channelpipeline

Channel的生命周期 ChannelUnregistered(已被创建还未注册) ChannelRegistered(已注册) ChannelActive(处于活动状态可以收发数据) ChannelInactive(未连接)

ChannelHandler的生命周期 handlerAdded(添加到ChannelPipeline时调用) handlerRemoved exceptionCaught

ChannelInBoundHandler处理入站数据及各种状态变化

​	前面那一堆+channelWritabilityChanged() 可写状态发生改变时调用+userEventTriggered() ChannelInboundHandler.fireUserTriggered()方法被调用时调用

​	重写channelRead()时 需要显示释放与池化的ByteBuf实例相关的内存 可以用ReferenceCountUtil.realse() (SimpleChannelInboundHandler可以自动释放)

ChannelOutBoundHandler处理出站数据并且允许拦截所有操作 可以按需推迟操作和事件

​	bind() connect() disconnect() close() deregister() read() flush() write()

两个默认实现类 ChannelInBoundHandlerAdapter ChannelOutBoundHandlerAdapter 还提供了isSharable() 是否被标注Sharable

ChannelPromise是ChannelFuture的一个子类 提供了setSuccess()和setFailure()等 使ChannelFuture不可变

ResourceLeakDetector 缓冲区的1%采样检测内存泄漏 检测级别:禁止 1%采样并报告发现的泄露(默认) 上一个+以及对应消息被访问的位置 类似前面 但是是每次

ChannelPipeline addFirst() addBefore() addAfter() addLast() remove() replace() 

​	触发事件 fireXXX()调用下一个ChannelInboundHandler的XXX xxx 调用下一个ChannelOutboundHandler的xxx

ChannelHandlerContext

​	ChannelHandlerContext代表了ChannelHandler和ChannelPipeline之间的关联上面那俩的方法全有 666 在加一些那啥的

共享同一个ChannelHandler 跨越多个Channel统计信息 要有@Sharable 必须线程安全(无状态)

异常处理 入站出站 ChannelFuture的子类ChannelPromise addListener()

### 第七章 EventLoop和线程模型

JDK的任务调度API 

1. newScheduledThreadPool指定延迟后运行 或者周期执行 使用corePoolSize参数来计算线程数
2. newSingleThreadScheduleExecutor 使用一个线程来执行被调度的任务

EventLoop调度任务 schedule和scheduleAtFixedRate()

异步传输	一个EventLoop多个Channel

阻塞传输	一个EventLoop一个Channel

### 第八章 引导

AbstractBootstrap<B extends AbstractBootstrap<B,C>, C extends Channel>

Bootstrap extends AbstractBpptstrap <Bootstrap, Channel>

ServerBootstrap extends AbstractBpptstrap <ServerBootstrap , ServerChannel>

Cloneable的 创建多个配置相同的Channel 返回的EventLoopGroup是浅拷贝 所以EventLoopGroup会被所有克隆的Channel之间共享

子类型是父类型的一个类型参数 因此可以返回到运行时实例的引用 以支持方法的链式调用

Bootstrap: group channel channelFactory(channel没有构造方法可以传一个工厂 会被bind调用) localAddress attr(指定新创建的属性值) handler clone remoteAddress connect bind(绑定Channel返回ChannelFuture 需要调用connect来建立连接)

ServerBootstrap: group channel channelFactory localAddress option(指定应用到新建的ServerChannel的ChannelConfig的ChannelOption 通过调用bind设置到Channel 调用bind后设置改变ChannelOption无效) childOption attr handler childHandler clone bind

ChannelOption 

AttributeKey<T> AttributeMap

DatagramChannel 无连接协议的 不会调用connect方法 只调用bind

关闭 EventLoopGroup.shutdownGracefully().sync() 阻塞直到完成

### 第九章 单元测试

EmbeddedChannel 

writeInbound(将入站消息写入 可通过readInbound读取) writeOutbound(将出站消息写入 readOutbound读取) 

finish(标记为完成 并且有可读的入站或出站数据返回true)

## 第二部分 编解码器 (10-11)

### 第十章 编解码器框架

解码器 ByteToMessage ReplayingDecoder MessageToMessageDecoder

decode 传入一个ByteBuf 和一个用于添加解码消息的List List不为空 则传入下一个ChannelInboundHandler

decodeLast 默认是实现调用了decode方法 当Channel状态变为非活动时 被调用一次 可以重写以实现生产LastHttpContent

一旦消息被解码或编码 就会被ReferenceCountUtil.release(message)释放 需要ReferenceCountUtil.retain(message)增加引用计数防止被释放

ReplayingDecoder 继承ByteToMessageDecoder 使用ReplayingDecoderByteBuf(readableBytes在内部被调用了) 字节不够抛出Signal

MessageToMessageDecoder 

TooLongFrameException ByteBuf.skipBytes然后抛出异常 被exceptionCaught捕获

编码器 MessageToByteEncoder MessageToMessageEecoder

encode 传入某类型 传出ByteBuf

没有last方法 因为解码器需要在Channel关闭后产生最后一个消息 而编码器不用

ByteToMessageCodec 

MessageToMessageCodec

CombinedChannelDuplexHandler

### 第十一章 预置的ChannelHandler和编解码器

SslHandler SSLEngine

HttpRequestEncoder   HttpRequestDecoder   HttpResponseEncoder   HttpResponseDecoder HttpClientCodec HttpServerCodec

HttpObjectAggregator 聚合 HttpContentCompressor数据压缩 HttpContentDecompressor解压

WebSocket真正的双向数据交换 以HttpServerCodec开始 然后WebSocketServerProtocolHandler

BinaryWebSocketFrame数据帧 二进制数据	TextWebSocketFrame数据帧 文本数据	ContinuationWebSocketFrame上一个的

CloseWebSocketFrame控制帧 一个CLOSE请求关闭的状态码以及原因 PingWebSocketFrame控制帧 请求PongWebSocketFrame 控制帧 响应前面那个 

IdleStateHandler空闲时间太长会触发一个IdleStateEvent 重写userEventTriggered来处理

ReadTimeoutHandler指定时间没有收到任何入站消息则抛出ReadTimeoutException并关闭Channel

WriteTimeoutHandler指定时间没有收到任何出站消息则抛出WriteTimeoutException并关闭Channel

DelimiterBasedFrameDecoder使用用户提供的分隔符来提取帧的通用解码器

LineBasedFrameDecoder 提取由\n或\r\n分隔的解码器

FixedLengthFrameDecoder根据构造方法提供的长度提取指定定长帧

LengthFieldBasedFrameDecoder根据编码进帧头部的长度值提取帧

FileRegion文件内容直接传输   

ChunkedWriteHandler 

序列化 Protocol Buffers

## 第三部分 网络协议 (12-13)

### 第十二章WebSocket



### 第十三章使用UDP广播事件



## 第四部分 案例研究 (14-15)

### 第十四章 案例研究一



### 第十五章 案例研究二



















## ------------------------------------------------------------------------------------------------------------------------------

| IO   |            |                                                              | 应用场景          |      |
| ---- | ---------- | ------------------------------------------------------------ | ----------------- | ---- |
| BIO  | 同步阻塞   | 有连接就启一个线程 这个连接不做事就造成了不必要的开销 可以通过线程池改善 | 连接数目小        |      |
| NIO  | 同步非阻塞 | 一个线程处理多个请求 注册到多路复用器(selector)              | 连接数目多 连接短 |      |
| AIO  | 异步非阻塞 | (1.7加入 没被广泛应用) 异步通道 Proactor 有效请求才启线程 (了解) | 链接数目多 连接长 |      |
|      |            |                                                              |                   |      |
|      |            |                                                              |                   |      |
|      |            |                                                              |                   |      |
|      |            |                                                              |                   |      |
|      |            |                                                              |                   |      |
|      |            |                                                              |                   |      |

## 
