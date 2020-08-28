# EventLoopGroup

## NioEventLoopGroup

- 构造方法形式多样，提供参数个数递增的构造函数，和有用的默认值

- 最终调用父类**MultithreadEventLoopGroup**的构造函数

  ```java
  super(nThreads, executor, selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject());//类似线程池？
  ```

- ```java
  public void setIoRatio(int ioRatio)//设置IO操作时间百分比.
  ```

- ```java
  public void rebuildSelectors()//解决JDK epoll 100% cpu bug.
  ```

- ```java
  //工厂方法具体由子类实现。构建并返回NioEventLoop
  @Override
  protected EventLoop newChild(Executor executor, Object... args) throws Exception {
      EventLoopTaskQueueFactory queueFactory = args.length == 4 ? (EventLoopTaskQueueFactory) args[3] : null;
      return new NioEventLoop(this, executor, (SelectorProvider) args[0],
                              ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2], queueFactory);
      }
  ```

## EpollEventLoopGroup

- 使用Epoll类检查是否支持epoll

- ```java
  //工厂方法具体由子类实现。构建并返回EpollEventLoop
  @Override
  protected EventLoop newChild(Executor executor, Object... args) throws Exception {
      EventLoopTaskQueueFactory queueFactory = args.length == 4 ? (EventLoopTaskQueueFactory) args[3] : null;
      return new EpollEventLoop(this, executor, (Integer) args[0],
                                ((SelectStrategyFactory) args[1]).newSelectStrategy(),
                                (RejectedExecutionHandler) args[2], queueFactory);
  }
  ```

- 其他同**NioEventLoopGroup**

  

## MultithreadEventLoopGroup

- ```java
  public abstract class MultithreadEventLoopGroup extends MultithreadEventExecutorGroup implements EventLoopGroup
  //MultithreadEventLoopGroup 既是MultithreadEventExecutorGroup，又是EventLoopGroop
  ```

- ```java
  private static final int DEFAULT_EVENT_LOOP_THREADS;
  //指定默认EVENT_LOOP数量为可用处理起个数乘2
  ```

- 实现EventLoopGroup接口的多个register方法原理：实际是在某个EventLoop上调用register方法

  ```java
  @Override
  public ChannelFuture register(ChannelPromise promise) {
      return next().register(promise);}
  ```

- 由于EventLoopGroup和MultithreadEventExecutorGroup都有next( )方法，所以需要Override  next( )方法以解决冲突，这里实际是调用父类next( )方法：

  ```java
  @Override
  public EventLoop next() {
  	return (EventLoop) super.next();}
  ```

- 覆盖了父类的newDefaultThreadFactory( )方法，默认线程工厂设置线程优先级**Thread.MAX_PRIORITY**：

  ```java
  @Override
  protected ThreadFactory newDefaultThreadFactory() {
      return new DefaultThreadFactory(getClass(), Thread.MAX_PRIORITY);}
  ```

- 构造函数调用父类的构造函数，主要控制了线程的数量，如果线程数为0则使用默认线程数量：

  ```java
  protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);}
  ```

  

## Executor

- ```java
  void execute(Runnable command);
  ```

## ExecutorService

```java
public interface ExecutorService extends Executor;
```

接口新增方法：

- ```java
  void shutdown();
  ```

- ```java
  List<Runnable> shutdownNow();
  ```

- ```java
  boolean isShutdown();
  ```

- ```java
  boolean isTerminated();
  ```

- ```java
  boolean awaitTermination(long timeout, TimeUnit unit)
  ```

- ```java
  Future<?> submit(Runnable task);
  ```

- ```java
  <T> Future<T> submit(Callable<T> task);
  ```

- ```java
  <T> Future<T> submit(Runnable task, T result);
  ```

- ```java
  <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
  ```

- ```java
  <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,                              long timeout, TimeUnit unit)
  ```

- ```java
  <T> T invokeAny(Collection<? extends Callable<T>> tasks)
  ```

- ```java
  <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                  long timeout, TimeUnit unit)
  ```

## ScheduledExecutorService

```java
public interface ScheduledExecutorService extends ExecutorService
```

接口新增方法：

- ```java
  public ScheduledFuture<?> schedule(Runnable command,
                                     long delay, TimeUnit unit);
  ```

- ```java
  public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                     long delay, TimeUnit unit);
  ```

- ```java
  public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                    long initialDelay,
                                                    long period,
                                                    TimeUnit unit);
  ```

  

- ```java
  public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                       long initialDelay,
                                                       long delay,
                                                       TimeUnit unit);
  ```

## EventExecutorGroup

```java
public interface EventExecutorGroup extends ScheduledExecutorService, 		                                                         Iterable<EventExecutor>
```

接口新增方法：

- ```java
  boolean isShuttingDown();
  ```

- ```java
  Future<?> shutdownGracefully();
  ```

- ```java
  Future<?> shutdownGracefully(long quietPeriod, long timeout, TimeUnit unit);
  ```

- ```java
  Future<?> terminationFuture();
  ```

- ```java
  EventExecutor next();
  ```

从**iterator**继承：

```java
@Override //Iterable
Iterator<EventExecutor> iterator();
```

从**ScheduledExecutorService**继承了四个**schedule**方法和三个 **submit**方法：

```java
@Override //ExecutorService
Future<?> submit(Runnable task);

@Override //ExecutorService
<T> Future<T> submit(Runnable task, T result);

@Override //ExecutorService
<T> Future<T> submit(Callable<T> task);

@Override //ScheduledExecutorService
ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);

@Override //ScheduledExecutorService
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);

@Override //ScheduledExecutorService
ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);

@Override //ScheduledExecutorService
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
```

当然还继承了其他父接口方法：

```java
void execute(Runnable command);
void shutdown();
List<Runnable> shutdownNow();
boolean isShutdown();
boolean isTerminated();
boolean awaitTermination(long timeout, TimeUnit unit);
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks);
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                              long timeout, TimeUnit unit);
<T> T invokeAny(Collection<? extends Callable<T>> tasks);
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit);
```



## AbstractEventExecutorGroup

```java
public abstract class AbstractEventExecutorGroup implements EventExecutorGroup
```

具体实现了**EventExecutorGroup**关于任务执行的全部方法，几乎全部使用 **next( )** 工厂方法返回的**EventExecutor**对象代理实现，**EventExecutor**同样实现了**EventExecutorGroup**接口：

```java
@Override
public Future<?> submit(Runnable task) {
    return next().submit(task);
}

@Override
public <T> Future<T> submit(Runnable task, T result) {
    return next().submit(task, result);
}

@Override
public <T> Future<T> submit(Callable<T> task) {
    return next().submit(task);
}

@Override
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    return next().schedule(command, delay, unit);
}

@Override
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) {
    return next().schedule(callable, delay, unit);
}

@Override
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay,
                                              long period, TimeUnit unit) {
    return next().scheduleAtFixedRate(command, initialDelay, period, unit);
}

@Override
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,
                                               long delay, TimeUnit unit) {
    return next().scheduleWithFixedDelay(command, initialDelay, delay, unit);
}

@Override
public <T> List<java.util.concurrent.Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    throws InterruptedException {
    return next().invokeAll(tasks);
}

@Override
public <T> List<java.util.concurrent.Future<T>> invokeAll(
    Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException {
    return next().invokeAll(tasks, timeout, unit);
}

@Override
public <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException {
    return next().invokeAny(tasks);
}

@Override
public <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    return next().invokeAny(tasks, timeout, unit);
}

@Override
public void execute(Runnable command) {
    next().execute(command);
}
```

还实现了如下方法，但都是依赖其他未实现方法：

```java
@Override
public Future<?> shutdownGracefully() {
    return shutdownGracefully(DEFAULT_SHUTDOWN_QUIET_PERIOD, DEFAULT_SHUTDOWN_TIMEOUT, TimeUnit.SECONDS);
}

public List<Runnable> shutdownNow() {
    shutdown();
    return Collections.emptyList();
}
```

未实现的方法有：

```java
void shutdown();//shutdownNow()依赖
boolean isShutdown();
boolean isTerminated();
boolean awaitTermination(long timeout, TimeUnit unit);
Iterator<EventExecutor> iterator();//迭代器
EventExecutor next();//任务执行相关方法依赖
//Future<?> shutdownGracefully();依赖
Future<?> shutdownGracefully(long quietPeriod, long timeout, TimeUnit unit);
boolean isShuttingDown();
Future<?> terminationFuture();

```

## MultithreadEventExecutorGroup

```java
public abstract class MultithreadEventExecutorGroup extends AbstractEventExecutorGroup
```

新增域：

```java
private final EventExecutor[] children;//Group管理的所有EventExecutor
private final Set<EventExecutor> readonlyChildren;//用于实现迭代器方法
//已关闭EventExecutor的计数值，等于children.length()，则children以经全部关闭
private final AtomicInteger terminatedChildren = new AtomicInteger();
private final Promise<?> terminationFuture = 
    new DefaultPromise(GlobalEventExecutor.INSTANCE);//用于接收termination通知
private final EventExecutorChooserFactory.EventExecutorChooser chooser;//选择器
```

新增方法：

- ```java
  //构造方法，初始化域
  protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                         EventExecutorChooserFactory chooserFactory, Object... args) {
      if (executor == null) {
          executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
      }
  	//初始换数组为指定大小
      children = new EventExecutor[nThreads];
  
      //循环创建EventExecutor填充数组
      for (int i = 0; i < nThreads; i ++) {
          boolean success = false;
          try {
              children[i] = newChild(executor, args);
              success = true;
          } catch (Exception e) {
              // TODO: Think about if this is a good exception type
              throw new IllegalStateException("failed to create a child event loop", e);
          } finally {
              if (!success) {
                  //某次创建失败，关闭所有EventExecutor
                  for (int j = 0; j < i; j ++) {
                      children[j].shutdownGracefully();
                  }
  				//等待所有线程优雅关闭shutdownGracefully
                  for (int j = 0; j < i; j ++) {
                      EventExecutor e = children[j];
                      try {
                          while (!e.isTerminated()) {
                              e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                          }
                      } catch (InterruptedException interrupted) {
                          // Let the caller handle the interruption.
                          Thread.currentThread().interrupt();
                          break;
                      }
                  }
              }
          }
      }
  	//初始换children成功，用它创建一个chooser，用于next()方法选择下一个EventExecutor
      chooser = chooserFactory.newChooser(children);
  	//EventExecutor操作完成时的监听器，如果所有children都已经关闭，则通知terminationFuture
      final FutureListener<Object> terminationListener = new FutureListener<Object>() {
          @Override
          public void operationComplete(Future<Object> future) throws Exception {
              if (terminatedChildren.incrementAndGet() == children.length) {
                  terminationFuture.setSuccess(null);
              }
          }
      };
  
      //为所有EventExecutor添加监听器
      for (EventExecutor e: children) {
          e.terminationFuture().addListener(terminationListener);
      }
  
      //创建不可修改的readonlyChildren
      Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>													(children.length);
      Collections.addAll(childrenSet, children);
      readonlyChildren = Collections.unmodifiableSet(childrenSet);
  }
  ```

- ```java
  //创建一个netty的默认线程池
  protected ThreadFactory newDefaultThreadFactory() {
  return new DefaultThreadFactory(getClass());
  }
  ```

- ```java
  //获取EventExecutor数量
  public final int executorCount() {
      return children.length;
  }
  ```

实现方法，全部实现了AbstractEventExecutorGroup未实现方法：

```java
    @Override
    public EventExecutor next() {
        return chooser.next();
    }

    @Override
    public Iterator<EventExecutor> iterator() {
        return readonlyChildren.iterator();
    }

    @Override
    public Future<?> terminationFuture() {
        return terminationFuture;
    }
//下面方法具有一个共同特点，都是遍历children，通过操作每一个EventExecutor状态实现
    @Override
    public Future<?> shutdownGracefully(long quietPeriod, long timeout, TimeUnit unit) {
        for (EventExecutor l: children) {
            l.shutdownGracefully(quietPeriod, timeout, unit);
        }
        return terminationFuture();
    }

    @Override
    @Deprecated
    public void shutdown() {
        for (EventExecutor l: children) {
            l.shutdown();
        }
    }

 	@Override
    public boolean isShuttingDown() {
        for (EventExecutor l: children) {
            if (!l.isShuttingDown()) {
                return false;
            }
        }
        return true;
    }

    @Override
    public boolean isShutdown() {
        for (EventExecutor l: children) {
            if (!l.isShutdown()) {
                return false;
            }
        }
        return true;
    }

    @Override
    public boolean isTerminated() {
        for (EventExecutor l: children) {
            if (!l.isTerminated()) {
                return false;
            }
        }
        return true;
    }

    @Override
    public boolean awaitTermination(long timeout, TimeUnit unit)
            throws InterruptedException {
        long deadline = System.nanoTime() + unit.toNanos(timeout);
        loop: for (EventExecutor l: children) {
            for (;;) {
                long timeLeft = deadline - System.nanoTime();
                if (timeLeft <= 0) {
                    break loop;
                }
                if (l.awaitTermination(timeLeft, TimeUnit.NANOSECONDS)) {
                    break;
                }
            }
        }
        return isTerminated();
    }
```



未实现方法，新增了一个工厂方法，具体由子类实现：

```java
//使用executor和参数创建EventExecutor
protected abstract EventExecutor newChild(Executor executor, Object... args);
```





## ThreadPerTaskExecutor

一个十分简单的执行器，使用线程工厂构造ThreadPerTaskExecutor对象，每次使用执行器去执行任务时，都会从线程工厂中创建一个新线程去执行任务

```java
public final class ThreadPerTaskExecutor implements Executor {
    private final ThreadFactory threadFactory;

    public ThreadPerTaskExecutor(ThreadFactory threadFactory) {
        if (threadFactory == null) {
            throw new NullPointerException("threadFactory");
        }
        this.threadFactory = threadFactory;
    }

    @Override
    public void execute(Runnable command) {
        threadFactory.newThread(command).start();
    }
}
```



父类Parent函数交互对象为A,子类Child override函数交互对象为B,则B必须是A的子类。

Group方法的实现总体比较简单，主要是通过调用child上的方法代理实现。

