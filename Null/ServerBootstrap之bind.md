# ServerBootstrap之bind()

当调用ServerbootStrap.bind()的一个重载方法后，服务器绑定到一个本地端口，本文跟着代码调试走一遍bind()流程。

## bind()

1、一个`bind()`重载方法，**所在类是代码第一行注释**

```java
//ServerbootStrap
public ChannelFuture bind(SocketAddress localAddress) {
    validate();
    return doBind(ObjectUtil.checkNotNull(localAddress, "localAddress"));
}
```

2、`doBind()`代码如下

```java
//ServerbootStrap
private ChannelFuture doBind(final SocketAddress localAddress) {
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    if (regFuture.cause() != null) {
        return regFuture;
    }

    if (regFuture.isDone()) {
        // At this point we know that the registration was complete and successful.
        ChannelPromise promise = channel.newPromise();
        doBind0(regFuture, channel, localAddress, promise);
        return promise;
    } else {
        //暂时不关心 !regFuture.isDone() 情况的处理。。。。。。
        return promise;
    }
}
```

> 在这段代码中，执行流程如下：
>
> 1、先调用`initAndRegister()`方法初始化channel，并将channel注册到一个EventLoop上。执行的结果作为一个ChannelFuture regFuture返回，其中包含了初始化后的channel。
>
> 2、如果regFuture操作成功完成，则创建一个ChannelPromise promise，该promise作为`doBind`方法的返回值，用来表示实际bind操作`doBind0(..., promise)`(具体参数见代码)的执行的结果。
>
> 下面分两个部分，一个是`initAndRegister()`，一个是`doBind0(..., promise)`

### initAndRegister()

1、`initAndRegister()`代码

```java
//AbstractBootstrap
final ChannelFuture initAndRegister() {
    Channel channel = null;
    try {
        channel = channelFactory.newChannel();
        init(channel);
    } 

    ChannelFuture regFuture = config().group().register(channel);
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) {
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    }
    
    return regFuture;
}

```

> 在上面代码中：
>
> 1、首先用工厂channelFactory创建了channel，然后调用 init(channel)初始化channel。
>
> 2、然后通过config().group()拿到给ServerBootstrap设置的EventloopGroup，调用其register方法将创建并初始化的channel注册到一个Eventloop上。

2、`init()`在AbstractBootstrap中是抽象方法，由子类Bootstrap实现，以下是ServerbootStrap的实现。

```java
//ServerbootStrap
void init(Channel channel) throws Exception {
    //为channel设置options
    final Map<ChannelOption<?>, Object> options = options0();
    synchronized (options) {
        setChannelOptions(channel, options, logger);
    }
	//为channel设置atrs
    final Map<AttributeKey<?>, Object> attrs = attrs0();
    synchronized (attrs) {
        for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
            @SuppressWarnings("unchecked")
            AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
            channel.attr(key).set(e.getValue());
        }
    }
	//拿到channel组合的pipeline
    ChannelPipeline p = channel.pipeline();

    //复制一份给子channel的配置，这些都是我们配置ServerbootStrap时传入的参数
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(0));
    }
    synchronized (childAttrs) {
        currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(0));
    }

    //给pipeline添加一个ChannelInitializer，其中的回调方法现在不会立即执行，执行时再分析其功能
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(final Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }

            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                        ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```

> `init(channel)`大的逻辑简单，虽然代码看起来挺长。

3、`config().group().register(channel)`实际调用的是MultithreadEventLoopGroup的register方法

```java
//MultithreadEventLoopGroup
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
```

> 很遗憾，register方法的实现并不是那么直接，而是通过EventLoop来实现。

4、next()

```java
//MultithreadEventLoopGroup
public EventLoop next() {
    return (EventLoop) super.next();
}
```

> 又调用的超类的next()

5、super.next()，MultithreadEventLoopGroup的超类是MultithreadEventExecutorGroup

```java
//MultithreadEventExecutorGroup
public EventExecutor next() {
    return chooser.next();
}
```

> 通过查看代码，
> chooser定义是：`chooser = chooserFactory.newChooser(children);`
> children定义是：`EventExecutor[] children`，它是一组EventExecutor。
> chooser.next()的作用就是按某种规则从children中选一个EventExecutor。
>
> 至此，可知第3步中，实际执行register操作的是一个EventLoop

6、继续第3步的register(channel)，执行register的是子类SingleThreadEventLoop

```java
//SingleThreadEventLoop
@Override
public ChannelFuture register(Channel channel) {
    return register(new DefaultChannelPromise(channel, this));
}

@Override
public ChannelFuture register(final ChannelPromise promise) {
    ObjectUtil.checkNotNull(promise, "promise");
    promise.channel().unsafe().register(this, promise);
    return promise;
}
```

> 执行register的对象是一个Eventloop实例，它被注入到了新创建的DefaultChannelPromise中，该promise也作为register操作的结果返回给register的调用者。
>
> 在第二个register方法中，promise.channel()拿到的实际就是之前创建的channel，由调用者传入，而unsafe()拿到的是该channel组合的内部类对象，用来替代channel执行一些操作，所以可以等价于执行channel自身的register方法。
>
> 下面看看channel自己是如何实现register方法的。

7、Channel的register，实际是它的Unsafe的register

```java
//AbstractChannel.AbstractUnsafe
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
  	//一些例行检查
    if (eventLoop == null) {
        throw new NullPointerException("eventLoop");
    }
    if (isRegistered()) {
        promise.setFailure(new IllegalStateException("registered to an event loop already"));
        return;
    }
    if (!isCompatible(eventLoop)) {
        promise.setFailure(
            new IllegalStateException("incompatible event loop type: " + eventLoop.getClass().getName()));
        return;
    }
	//将传入的EventLoop绑定到该Channel
    AbstractChannel.this.eventLoop = eventLoop;
	//调试时if条件不成立，走else
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else {
        try {
            //将实际执行register的操作register0作为eventloop的一个任务执行，用promise通知执行结果
            eventLoop.execute(new Runnable() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            logger.warn(
                "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                AbstractChannel.this, t);
            closeForcibly();
            closeFuture.setClosed();
            safeSetFailure(promise, t);
        }
    }
}
```

> 将传入的EventLoop绑定到该Channel
>
> 将实际执行register的操作register0作为eventloop的一个任务执行，用promise通知执行结果
>
> 至此，initAndRegister()调用结束，绑定的eventloop异步执行实际register0的操作，并通知initAndRegister()返回的regFuture操作的结果。

### register0

1、register0主要代码如下：

```java
//AbstractChannel

private void register0(ChannelPromise promise) {
    try {
        boolean firstRegistration = neverRegistered;
        //1、将本Channel聚合的ServiceChannel注册到EventLoop的Selector上，对0事件感兴趣
        doRegister();
        neverRegistered = false;
        registered = true;
        //2、调用第一个handler的handlerAdded()方法，即在init(channel)时设置的ChannelInitializer
        pipeline.invokeHandlerAddedIfNeeded();
		//3、通知ChannelFuture，bond操作成功
        safeSetSuccess(promise);
        //4、
        pipeline.fireChannelRegistered();
        // Only fire a channelActive if the channel has never been registered. This prevents firing
        // multiple channel actives if the channel is deregistered and re-registered.
        if (isActive()) {
            if (firstRegistration) {
                pipeline.fireChannelActive();
            } else if (config().isAutoRead()) {
                // This channel was registered before and autoRead() is set. This means we need to begin read
                // again so that we process inbound data.
                //
                // See https://github.com/netty/netty/issues/4805
                beginRead();
            }
        }
    } catch (Throwable t) {
        // Close the channel directly to avoid FD leak.
        closeForcibly();
        closeFuture.setClosed();
        safeSetFailure(promise, t);
    }
}

```

> 1、doRegister()将channel聚合的ServiceChannel注册到EventLoop的Selector上
>
> 2、调用第一个handler的handlerAdded()方法
>
> 3、通知ChannelFuture，bond操作成功
>
> 现面跟着代码一步一步看

2、doRegister()

```java
//AbstractNioChannel

protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            selectionKey = javaChannel()
                .register(eventLoop().unwrappedSelector(), 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                //selectNow（）去掉缓存的已被取消的selectionKey，再试一次？
                eventLoop().selectNow();
                selected = true;
            } else {
                //这里代码永远不能执行，否则就是jdk有bug
                throw e;
            }
        }
    }
}

```

> javaChannel()拿到该channel聚合的SelectableChannel，实际就是JDK NIO的注册操作
>
> eventLoop().unwrappedSelector()拿到的是JAVA NIO的Selector

3、pipeline.invokeHandlerAddedIfNeeded()，调用handlerAdded()方法

```java
//DefaultChannelPipeline

final void invokeHandlerAddedIfNeeded() {
    assert channel.eventLoop().inEventLoop();
    if (firstRegistration) {
        firstRegistration = false;
        callHandlerAddedForAllHandlers();
    }
}
```

> 继续调用callHandlerAddedForAllHandlers();

4、callHandlerAddedForAllHandlers();

```

```

