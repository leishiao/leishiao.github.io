# netty-read 总结

1、可以继承现有类或抽象类去实现接口

2、如果接口层次深，可以从继承某一个合适的高层次接口的抽象类开始，逐层兑现个层次接口

3、外部线程可以使用一个空任务去唤醒EventLoop因为BlockingQueue的take方法而阻塞的内部执行线程：

```java
private static final Runnable WAKEUP_TASK = new Runnable() {
    @Override
    public void run() {
        // Do nothing.    
    }
};
```

4、AtomicIntegerFieldUpdater

```java
    private static final AtomicIntegerFieldUpdater<SingleThreadEventExecutor> STATE_UPDATER =
            AtomicIntegerFieldUpdater.newUpdater(SingleThreadEventExecutor.class, "state");
```

5、someInterfaceGroup和someInterface的关系

```java
someInterface extends someInterfaceGroup
```

6、每个具体子类都应该实现接口，而不是仅继承父类

```java
interface Employee{}
class Worker implements Employee{}
class ConcreteManager implements Employee{}//不好，违背面向接口编程

interface Manager extends Employee{}//推荐
class ConcreteManager extends Worker implements Manager{}//继承worker来实现Manager接口
```

