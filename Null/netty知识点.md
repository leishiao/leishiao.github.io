# netty知识点

架构方法和设计原则是：每个小点都和它的技术性内容一样重要，穷其精妙。
我们已经从漫长的痛苦经历中学到：直接使用底层的API暴露了复杂性，并且引入了对往往供不应求的技能的关键性依赖
Netty使你可以专注于自己真正感兴趣的——你的应用程序的独一无二的价值。
本质上，一个既是异步的又是事件驱动的系统会表现出一种特殊的、对我们来说极具价值的行为：它可以以任意的顺序响应在任意的时间点产生的事件。
非阻塞网络调用使得我们可以不必等待一个操作的完成。异步方法会立即返回，并且在它完成时，会直接或者在稍后的某个时间点通知用户。
选择器使得我们能够通过较少的线程便可监视许多连接上的事件。
Channel 是 Java NIO 的一个基本构造。它代表一个到实体的开放连接，如读操作和写操作。
可以把 Channel 看作是传入（入站）或者传出（出站）数据的载体。因此，它可以被打开或者被关闭，连接或者断开连接。
一个回调其实就是一个方法，回调在广泛的编程场景中都有应用，而且也是在操作完成后通知相关方最常见的方式之一。
Future 提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。
ChannelFuture使得我们能够注册一个或者多个ChannelFutureListener实例。
监听器的回调方法 operationComplete()，将会在对应的操作完成时被调用。监听器可以判断该操作是成功地完成了还是出错了。
由ChannelFutureListener提供的通知机制消除了手动检查对应的操作是否完成的必要。
每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture；也就是说，它们都不会阻塞，Netty 完全是异步和事件驱动的。
Netty 使用不同的事件来通知我们状态的改变或者是操作的状态。这使得我们能够基于已经发生的事件来触发适当的动作。记录日志；数据转换；流控制；应用程序逻辑。
由入站数据或者相关的状态更改而触发的事件包括：由入站数据或者相关的状态更改而触发的事件包括：数据读取；用户事件；错误事件。
出站事件是未来将会触发的某个动作的操作结果，这些动作包括：打开或者关闭到远程节点的连接；将数据写到或者冲刷到套接字。
可以认为每个 ChannelHandler 的实例都类似于一种为了响应特定事件而被执行的回调。
Netty的异步编程模型是建立在Future和回调的概念之上的， 而将事件派发到ChannelHandler的方法则发生在更深的层次上。

Netty 的 Channel 接口所提供的 API，大大地降低了直接使用 Socket 类的复杂性。
Channel 也是拥有许多预定义的、专门化实现的广泛类层次结构的根，EmbeddedChannel ；LocalServerChannel ；NioDatagramChannel ；NioSctpChannel ；NioSocketChannel 。
所有属于同一个 Channel 的操作都被保证其将以它们被调用的顺序被执行。

一个 EventLoopGroup 包含一个或者多个 EventLoop；
一个 EventLoop 在它的生命周期内只和一个 Thread 绑定；
所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理；
一个 Channel 在它的生命周期内只注册于一个 EventLoop；
一个 EventLoop 可能会被分配给一个或多个 Channel。多个channel共享一个EventLoop
一个给定 Channel 的 I/O 操作都是由相同的 Thread 执行的，实际上消除了对于同步的需要。

当 Channel 被创建时，它会被自动地分配到它专属的 ChannelPipeline。
当ChannelHandler被添加到ChannelPipeline时，它将会被分配一个ChannelHandlerContext，其代表了 ChannelHandler 和 ChannelPipeline 之间的绑定。虽然这个对象可以被用于获取底层的 Channel，但是它主要还是被用于写出站数据。

在Netty中，有两种发送消息的方式。你可以直接写到Channel中，也可以 写到和ChannelHandler相关联的ChannelHandlerContext对象中。前一种方式将会导致消息从ChannelPipeline 的尾端开始流动，而后者将导致消息从 ChannelPipeline中的下一个 ChannelHandler 开始流动。

ChannelHandler 的典型用途包括：
1、将数据从一种格式转换为另一种格式；
2、提供异常的通知；
3、提供 Channel 变为活动的或者非活动的通知；
4、提供当 Channel 注册到 EventLoop 或者从 EventLoop 注销时的通知；
5、提供有关用户自定义事件的通知。

![1567240504250](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1567240504250.png)