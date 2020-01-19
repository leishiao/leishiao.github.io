## CHAPTER 5 Building Blocks

Where practical, delegation is one of the most effective strategies for creating thread-safe classes: just let existing thread-safe classes manage all the state.

This chapter covers the most useful concurrent building blocks and some patterns for using them to structure concurrent applications.

### 5.1 Synchronized collections

The *synchronized collections classes* include Vector and HashTable, and the synchronized wrapper classes.

#### 5.1.1 Problems with synchronized collections

The synchronized collections are thread-safe, but you may sometimes need to use additional client-side locking to guard compound actions. Compound actions on collections include iteration, navigation and conditional actions.

#### 5.1.2 Iterators and ConcurrentModificationException

The standard way to iterate a collection is with an Iterator (or for-each), but using iterator does not obviate the need to lock the collection during iteration.

The iterators returned by synchronized collections is not designed to deal with concurrent modification, they are *fail-fast*, meaning that if they detect that if the collection has changed since the iteration began, they throw ConcurrentModificationException. They are implemented by associating a modification count with the collection: if the modification count changes during iteration, `hasNext` or `next` throw ConcurrentModificationException.

An alternative to locking the collection during iteration is to copy the collection and iterate the copy instead.

#### 5.1.3 Hidden iterators

Iteration is also indirectly invoked by the collection's `hashCode` and `equals` methods, which may be called if the collection is used as an element or key of another collection. Similarly, the `containsAll`, `removeAll`, `retainAll` methods, as well as the constructors that take collections as arguments, also iterate the collection.

> Just as encapsulating an object's state makes it easier to preserve the invariants, encapsulating its synchronization makes it easier to enforce its synchronization policy.

### 5.2 Concurrent collections

> Replacing synchronized collections with concurrent collections can offer dramatic scalability improvements with little risk.

**ConcurrentHashMap**: a replacement for synchronized hash-based Map implementations.

**CopyOnWriteArrayList**: a replacement for synchronized List implementations.

**CopyOnWriteArraySet**: a replacement for synchronized Set implementations.

Queue: A queue is intended to hold elements temporarily while they wait processing.

**ConcurrentLinkedQueue**: a traditional FIFO queue.

**PriorityQueue**: a (non concurrent) priority ordered queue.

Queue operations do not block, if the queue is empty, the retrieval operation returns null.

Queue admits more concurrent implementations than List without its random-access.

**BlockingQueue** extends Queue to add blocking insertions and retrieval operations.

**ConcurrentSkipListMap**: a replacement for synchronized SortedMap.

**ConcurrentSkipListSet**: a replacement for synchronized SortedSet.

#### 5.2.1 ConcurrentHashMap

ConcurrentHashMap is a hash-based Map like hashMap, but it uses an entirely different locking strategy that offers better concurrency and scalability. Instead of synchronizing every method on a common lock, restricting access to a single thread at a time, it uses a finer-grained locking mechanism called *lock striping* to allow a greater degree of shared access.

ConcurrentHashMap, along with other concurrent collections, provides iterators that do not throw ConcurrentModificationException, thus eliminating the need to lock a collection during iteration. Iterator returned by ConcurrentHashMap is *weakly consisted* instead of *fail-fast*.

#### 5.2.2 Additional atomic Map operations

Since ConcurrentHashMap can not be locked for exclusive access, we cannot use client-side locking to create new atomic compound operations such as *put-if-absent*. Instead, a number of compound operations such as *put-if-absent*, *remove-if-equal*, and *replace-if-equal* are implemented as atomic operations and specified by the ConcurrentMap interface.

#### 5.2.3 CopyOnWriteArrayList / CopyOnWriteArraySet

The copy-on-write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published no further synchronization is required when accessing it. They implement mutability by creating and publishing a new copy of the collection every time it is modified.

Iterators from copy-on-write collections retains a reference the backing array that was current at the start of iteration and does not throw ConcurrentModificationException.

### 5.3 Blocking queues and producer-consumer pattern

Blocking queues provide blocking *put* and *take* method as well as the timed equivalents *offer* and *poll*. If a queue is full, *put* blocks until space becomes available; if the queue is empty, *take* blocks until an element is available.

A *producer-consumer* design separates the  identification of work to be done with the execution of that work by placing work items on a 'to do' list for later processing. BlockingQueue simplifies the *producer-consumer* design pattern.

One of the most common producer-consumer designs is a thread pool coupled with a work queue; this pattern is embodied in the Executor task execution framework.

Blocking queues also provide an *offer* method, which returns a failure status value if the item cannot be enqueued. This enables you create more flexible policies for dealing with overload, such as reducing the number of producer threads.

> Bounded queues are a powerful resource management tool for building reliable applications: they make your program more robust to overload by throttling activities that threaten to produce more work than can be handled.

It is tempting to assume that the consumers will always keep up, so that you need not place any bounds on the size of work queues, but this is a prescription for rearchitecting your system later.  Build resource management into your design early using blocking queues, it is a lot easier to do this up front than to retrofit it later.

The class library contains several implementations of BlockingQueue. **LinkedBlockingQueue** and **ArrayBlockingQueue** are FIFO queues. **PriorityBlockingQueue** is a priority-ordered queue, it is useful when you want to process elements other than FIFO. 

**SynchronousQueue**, is not really a queue at all, it maintains no storage space for queued elements. Instead, it maintains a list of threads waiting to enqueue or dequeue an element. Since a SynchronousQueue has no storage capacity, *take* and *put* will block unless another thread is already waiting to participate in the handoff. 

#### 5.3.2 Serial thread confinement

For mutable objects, producer-consumer designs and blocking queues  facilitate *serial thread confinement* for handing off ownership of objects from producers to consumers.

Object pools exploit serial thread confinement, 'lending' an object to the requesting thread, the ownership can be transferred from thread to thread.

One could also use other publication mechanisms for transferring the ownership of a mutable object, it is necessary to ensure that only one thread receives the object being handed off. BlockingQueue makes this easy; with a little more work, it could also done with the atomic *remove* method of ConcurrentMap or the *compareAndSet* method of AtomicReference.

#### 5.3.3 Deques and work stealing

Java 6 also adds another collection types, Deque and BlockingDeque, that extend Queue and BlockingQueue. A Deque is a double-ended queue that allows efficient insertion and removal from both the tail and head. Implementations include **ArrayDeque** and **LinkedBlockingDeque**.

Just as blocking queues lend themselves to the producer-consumer pattern, deques lend themselves to a relative pattern called *work stealing*.

### 5.4 Blocking and interruptible methods

When a thread blocks, it is usually suspended and placed in one of the blocked thread state (BLOCKED, WAITTING or TIMED_WAITTING). A blocked thread must wait for an event that is beyond its control before it can proceed, such as the I/O completes, the lock becomes available, or the external computation finishes. When the external event occurs, the thread is placed back in RUNNABLE state and becomes eligible again for scheduling. 

When a method can throw InterruptedException, it is telling you that it is a blocking method, and further that if it is interrupted, it will take efforts to stop blocking early.

Interruption is a cooperative mechanism, the most sensible use for interruption is to cancel an activity. Each thread has a boolean property that represents the interrupted status, interrupting a thread sets this status.

When your code calls a blocking method, your method is a blocking method too, and must has a plan for responding interruption. For library code, there are basically two choices:

**Propagate the InterruptedException.**  This is always the most sensible policy if you can get away with it, just propagate it to your caller.

**Restore the interrupt.** Sometimes you cannot throw InterruptedException, for instance when your code is part of a Runnable. In these situations, you must catch it and restore the interrupted status by calling *interrupt* on the current thread, so that code higher up the call stack can see that a interrupt is issued.

You should not catch InterruptedException and do nothing in response. The only situation in which it is acceptable to swallow an interrupt is when you are extending Thread and control all the code higher up on the call stack.

###  5.5 Synchronizers

A synchronizer is an object that coordinates the control flow of threads based on its state. **Blocking queues** can act as synchronizer, other type of synchronizers include **semaphores**, **barriers**, and **latches**. 

You can create your own. 

All synchronizer share certain structural properties: they encapsulate state that determines whether threads arriving at the synchronizer should be allowed to pass or forced to wait, provide methods to manipulate that state, and provide methods to wait efficiently for the synchronizer to enter the desired state.

#### 5.5.1 Latches

A latch is a synchronizer that can delay the progress of threads until it reaches its terminal state.

A latch acts as a gate: until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass. Once the latch reaches the terminal state, it cannot change state again, so it remains open forever. Laches can be used to ensure that certain activities can not proceed until other one-time activities complete.

#### 5.5.2 FutureTask

FutureTask also acts like a latch. FutureTask implements Future, which describes an abstract result-bearing computation. A computation presented by FutureTask is implemented with Callable, and can be one of three states: waiting to run, running, or completed. Computation subsumes all the ways a computation can complete, including normal completion, cancellation and exception. Once a FutureTask enters the completion state, it stays in that state forever.

#### 5.5.3 Semaphores

Counting semaphores are used to control the number of activities that can access a certain resource or perform a given action at the same time. Counting semaphores can be used to implement resource pools or to impose a bound on a collection.

#### 5.5.4 Barriers

Barriers are similar to latches in that they block a group of threads until some event has occurred. The key difference is that with a barrier, all threads must come together at a *barrier point* at the same time in order to proceed. Latches are for waiting for events, barriers are for waiting for other threads. 