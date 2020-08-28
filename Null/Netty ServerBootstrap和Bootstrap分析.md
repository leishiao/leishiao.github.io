# Netty ServerBootstrap和Bootstrap分析

## 1、类图

### 1.1 XBootstrap类图

在netty中，使用ServerBootstrap引导服务器，用Bootstrap引导客户端，两者都继承共同的抽象父类AbstractBootstrap。如图：

![bootstrap类图](C:\Users\leixiao\Desktop\bootstrap解析\bootstrap类图.png)

**AbstractBootstrap**聚合了有6个实例域：

| 名称          | 关系     | 作用                                  |
| ------------- | -------- | ------------------------------------- |
| group         | 单向关联 | 驱动netty执行的EventLoopGroup         |
| channelFacory | 单向关联 | 用于在需要的时候创建指定的Channel实例 |
| localAddress  | 单向关联 | 作为服务器启动时绑定的SocketAddress   |
| options       | 单向关联 | 父channel的选项（具体用途不明）       |
| attrs         | 单向关联 | 父channel的属性（具体用途不明）       |
| handler       | 单向关联 | 引导服务器时指定的handler             |

**ServerBootstrap**在继承了AbstractBootstrap实例域的基础上，又增加了以`child`开头的4个实例域，用于配置子Channel的EventLoop，属性，选项和handler。**Bootstrap**增加了remoteAddress域，因为客户端需要连接到服务器的远程地址。logger用于记录日志。

AddressResolverGroup类型变量需要的时候再看。（具体用途不明）

XBootstrap中提供了设置或创建这些实例域的方法，一些实例域也提供了公有的获取方法。完整类图如下：

![AllBootstrap](C:\Users\leixiao\Desktop\bootstrap解析\AllBootstrap.png)



### 1.2 XBootstrapConfig类图

AbstractBootstrapConfg、ServerBootstrapConfg和BootstrapConfig的类图如下：

![bootstrapconfig](C:\Users\leixiao\Desktop\bootstrap解析\bootstrapconfig.png)

在抽象父类AbstractBootstrapConfg中，通过构造方法**聚合**了一个AbstractBootstrap类型的变量，而子类将聚合的对象更加具体化。从XBootstrapConfig的方法中不难看出，XBootstrapConfig的作用就是从所**聚合**的XBootstrap中获取实例域。XBootstrap和XBootstrapConfig是**双向关联**关系，你中有我，我中有你。

### 1.3 **Future/Promise异步模型** 

>  在并发编程中，我们通常会用到一组非阻塞的模型：Promise，Future 和 Callback。其中的 Future 表示一个可能还没有实际完成的异步任务的结果，针对这个结果可以添加 Callback 以便在任务执行成功或失败后做出对应的操作，而 Promise 交由任务执行者，任务执行者通过 Promise 可以标记任务完成或者失败。 可以说这一套模型是很多异步非阻塞架构的基础。Netty 4中正提供了这种Future/Promise异步模型。
>
> Netty在源码上大量使用了Future/Promise模型，在Netty里面也是这样定义的：
>
> - Future接口定义了isSuccess(),isCancellable(),cause(),这些判断异步执行状态的方法。（read-only）
>
> - Promise接口在extneds future的基础上增加了setSuccess(), setFailure()这些方法。（writable）

#### Netty的Future/Promise接口框架

![Promise](C:\Users\leixiao\Desktop\bootstrap解析\Promise.png)

**ChannelFuture接口**

![ChannelFuture](C:\Users\leixiao\Desktop\bootstrap解析\ChannelFuture.png)

Netty中的Future接口继承自java的Future接口并且名称相同。在原有方法的基础上，增加了丰富的获取Future状态的方法，设置和删除监听器的方法，阻塞等待执行结果的方法。有些方法返回类型为Future，是为了支持链式调用。

ChannelFuture接口是绑定了Channel的Future，增加了两个方法：

| 方法名称  | 作用                                                        |
| --------- | ----------------------------------------------------------- |
| channel() | 放回绑定的Channel                                           |
| isVoid()  | 查询当前ChannelFuture是否是void，如果是，有些方法就不能执行 |

并且**将支持链式调用的方法的返回类型子类化**，Future改为ChannelFuture。

纵观类图中的所有方法，全都是获取执行状态或者等待结果的方法，没有设置Futrue状态和结果的相关的方法，因此Future是只读的（**read-only**）。

**Promise接口**

![Promise接口](C:\Users\leixiao\Desktop\bootstrap解析\Promise接口.png)

Promise接口同样是继承自Future，在Future获取状态方法的的基础上，还增加修改状态和设置结果的方法。

因此Promise是可读可写的（writeable）Future。

**ChannelPromise接口**

为了使ChannelFuture这个绑定了Channel的Future变成可写的，ChannelPromise接口继承了ChannelFuture和Promise两个接口，ChannelPromise接口可读可写，是netty实际使用较多的接口类型。

![ChannelPromise2](C:\Users\leixiao\Desktop\bootstrap解析\ChannelPromise2.png)

**DefautChannelPromise**是ChannelPromise的实现类，实现层次结构如下：

![DefautChannelArch](C:\Users\leixiao\Desktop\bootstrap解析\DefautChannelArch.png)

DefaultChannelPromise实现ChannelPromise接口，**并不是直接实现该接口的所有方法**，而是通过一个继承层次结构逐步实现。

AbstractFuture 实现了Future接口的`get()`方法。

DefaultPromise实现了Promise接口剩下的所有方法。

DefaultChannelPromise实现了有关Channel的方法，和FlushCheckPoint接口的方法，并且**覆盖了支持链式调用的方法**。

![DefaultChannelPromise](C:\Users\leixiao\Desktop\bootstrap解析\DefaultChannelPromise.png)

未完待续......