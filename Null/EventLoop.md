# EventLoop



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

## EventExecutor

```java
public interface EventExecutor extends EventExecutorGroup
```

接口新增方法：

- ```java
  EventExecutorGroup parent();
  
  boolean inEventLoop();
  
  boolean inEventLoop(Thread thread);
  
  <V> Promise<V> newPromise();
  
  <V> ProgressivePromise<V> newProgressivePromise();
  //Create a new {@link Future} which is marked as succeeded already.
  <V> Future<V> newSucceededFuture(V result);
  //Create a new {@link Future} which is marked as failed already.
  <V> Future<V> newFailedFuture(Throwable cause);
  ```

## OrderedEventExecutor

```java
/**
 * Marker interface for {@link EventExecutor}s that will process all submitted tasks in  an ordered / serial fashion.
 */
public interface OrderedEventExecutor extends EventExecutor
```

## EventLoop

```java
public interface EventLoop extends OrderedEventExecutor, EventLoopGroup
```













