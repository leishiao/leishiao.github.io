## RabbitMQ Java 客户端主要类和接口：

Channel:提供大部分的协议方法操作

Connection：代表一个AMQP连接

ConnectionFactory：用于创建Connection实例

Consumer：代表一个消费者

DefaultConsumer：广泛使用的消费者的基类

BasicProperties：消息属性，消息元数据

BasicProperties.Builder：BasicProperties的建造者

Connection 用来

1，open channels

2，register **connection lifecycle event** handlers

3，close connections 

ConnectionFactory 用来设置连接参数

## 连接到RabbitMQ

使用ConnectionFactory创建连接：

```java
ConnectionFactory factory = new ConnectionFactory();
// "guest"/"guest" by default, limited to localhost connections
factory.setUsername(userName);
factory.setPassword(password);
factory.setVirtualHost(virtualHost);//default "/"
factory.setHost(hostName);//default "localhost"
factory.setPort(portNumber);//defualt 5672,5671(TLS)

Connection conn = factory.newConnection();
```

## 与RabbitMQ断开连接

简单的关闭channel和connecton即可

```java
channel.close();
conn.close();
```

关闭channel是一种好习惯，但是channel并不强制要求关闭，因为低层的connection关闭的时候，所有的channel会自动关闭。

## Connection 和 Channel 的存活时间

Connections 可以长时间存活，低层协议为长时间运行的connections设计和优化。因此不必为每个操作（比如发布一个消息）都建立一个新的connection。

Channels也可以长时间存活，但是许多错误会导致channel被关闭，channel的存活时间可能会比它的底层connection短。重用channel，而不是为每个操作都打开一个channel然后关闭它。

Channel-level exceptions 会导致Channel被关闭，被关闭的Channel不能再使用

## 关闭协议

### 关闭流程概述

connection 和 channel 使用相同的方法处理 network failure, internal failure, and explicit local shutdown.

connection 和 channel 有如下生命周期状态：

- **open**: the object is ready to use
- **closing**: the object has been explicitly notified to shut down locally, has issued a shutdown request to any supporting lower-layer objects, and is waiting for their shutdown procedures to complete
- **closed**: the object has received all shutdown-complete notification(s) from any lower-layer objects, and as a consequence has shut itself down

connection s和 channels 终归会到closed状态，不论是什么原因导致关闭，比如应用程序要求关闭，内部库错误，远程网络请求或网络错误。

connection 和 channel 对象有以下处理有关shutdown的方法：

- addShutdownListener(ShutdownListener listener) 

- removeShutdownListener(ShutdownListener listener)

用来管理listeners，当对象转变为closed状态时，这些listener会被触发。

- getCloseReason()用来查看关闭的原因

- getCloseReason()测试对象是否是open状态


- close(int closeCode, String closeMessage)通知对象关闭

listener的简单使用

```java
connection.addShutdownListener(new ShutdownListener() {
    public void shutdownCompleted(ShutdownSignalException cause)
    {
        ...
    }
});
```

### 获取某次shutdown的信息

通过**getCloseReason()**或者listener方法的参数获获得**ShutdownSignalException**对象，这个对象包含关闭原因的信息。

**isHardError()**方法判断 whether it was a connection or a channel error，

**getReason()**returns information about the cause, in the form an AMQP method - eitherAMQP.Channel.Close or AMQP.Connection.Close (or null if the cause was some exception in the library, such as a network communication failure, in which case that exception can be retrieved with **getCause()**）

## 高级连接设置

### 使用多个连接地址

可以向**newConnection()**方法传递**Address**数组，**Address**类是包含host和port的简单类。例如：

```java
Address[] addrArr = new Address[]{ new Address(hostname1, portnumber1)
                                 , new Address(hostname2, portnumber2)};
Connection conn = factory.newConnection(addrArr);
```

这样会依次尝试连接Address，直到某次连接成功。



## 从网络故障中自动恢复

### Connection自动恢复

clients和RabbitMQ节点之间的网络连接可能会出错。Java client支持connections和topology（queues, exchanges, bindings, and consumers）的自动恢复。

Connection和Channel自动恢复流程如下：

- 重新连接
- 恢复连接上的listeners
- 重新打开channels
- 恢复channel上的listener
- 恢复channel上basic.qos setting, publisher confirms and transaction settings

每个channel的Topology 恢复流程如下：

- 重新declare exchanges 
- 重新declare queues
- 恢复所有的bindings
- 恢复所有的consumers

Java客户端4.0版本以后，自动恢复是默认打开的，也可以在代码中打开：

```java
factory.setAutomaticRecoveryEnabled(true);
```

如果恢复因为异常失败，比如RabbitMQ节点依然不可达，会在固定时间间隔和重试，可以设置时间间隔：

```java
// attempt recovery every 10 seconds
factory.setNetworkRecoveryInterval(10000);
```

如果connection设置的是一组Address，这些Address会被打乱顺序，然后依次尝试。

### 什么时候Connection recovery被触发？

自动connection recovery 会被以下事件触发：

- connection的 I/O loop抛出一个 I/O exception

- 一个socket read 操作超时

- 检测到**heartbeats** 丢失

- connection的 I/O loop抛出的其他异常

Channel-level exceptions 不会触发任何恢复流程。

### 恢复监听器 Recovery Listeners

可以在connections 和 channels上设置多个Recovery Listeners，有两个相关方法：

- addRecoveryListener

- removeRecoveryListener

### 对Publishing的影响

当connection断开时，使用Channel.basicPublish发布的消会丢失。要确保消息到达目标RabbitMQ程序，需要使用 **Publisher Confirms**，and account for connection failures.

### 故障检测和恢复的局限性

当connection中断或丢失后，需要一段时间才能检测到，因此出现一个窗口期，在这期间通信两端都不知道connection故障。这段时间内消息会向往常一样写到TCP socket中，这些消息能否传递到broker，只能用 **Publisher Confirms**保证：AMQP 0-9-1中发布消息被设计成完全异步的。



当 socket 或  I/O 操作错误被 connection 检测到，在指定时间延迟后，自动恢复开始执行，默认5s。这个设计假设,尽管大多数的网络故障都是短暂的，但也不会立即消失。

Connection recovery 默认在相同的时间间隔后重试，直到成功建立一个new connection .

当connection处在**正在恢复**状态时，用它打开的channel执行任何publish操作都将抛出异常，客户端不会缓存这些消息，开发人员需要追踪这些消息，并在恢复成功时republish.不能消息丢失的发布者应当使用**Publisher Confirms**。

被关闭的channel不能被恢复，即使在connection recovery介入后，包括调用close方法关闭的和 channel-level exception异常导致关闭的。

### 手动Acknowledgements 和Automatic Recovery

connection恢复后，RabbitMQ会重新设置所有channel的delivery tags。

使用手动确认和自动恢复的应用程序必须能够处理重复的消息投递。

### Channels Lifecycle 和Topology Recovery

自动 connection recovery 对开发人员来说是尽可能透明的，这就是为什么尽管经历了多次连接故障和自动恢复后，Channel实例仍然可以保持相同的原因。从技术上讲，当自动恢复打开的情况下，创建的Channel实例是作为代理人和装饰者的：他们将工作委派给一个正在的Channel并围绕它实现恢复机制。这就是为什么在Channel创建了一些资源 (queues, exchanges, bindings)之后不应该关闭它的原因，否则这些资源的恢复将失败。Instead, leave creating channels open for the life of the application.

### Unhandled Exceptions

connection, channel, recovery, and consumer lifecycle的异常被委托给exception handler。Exception handler实现了**ExceptionHandler**接口，默认使用的 DefaultExceptionHandler 打印异常的详细信息。

```java
ConnectionFactory factory = new ConnectionFactory();
cf.setExceptionHandler(customHandler);
```



