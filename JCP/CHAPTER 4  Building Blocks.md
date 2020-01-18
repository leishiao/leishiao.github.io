## CHAPTER 5 Building Blocks

Where practical, delegation is one of the most effective strategies for creating thread-safe classes: just let existing thread-safe classes manage all the state.

This chapter covers the most useful concurrent building blocks and some patterns for using them to structure concurrent applications.

### 5.1 Synchronized collections

The *synchronized collections classes* include Vector and HashTable, and the synchronized wrapper classes.

#### 5.1.1 Problems with synchronized collections

The synchronized collections are thread-safe, but you may sometimes need to additional client-side locking to guard compound actions. Compound actions on collections include iteration, navigation and conditional actions.

#### 5.1.2 Iterators and ConcurrentModificationException

The standard way to iterate a collection is with an Iterator (or for-each), but using iterator does not obviate the need to lock the collection during iteration.

The iterators returned by synchronized collections is not designed to deal with concurrent modification, they are *fail-fast*, meaning that if they detect that if the collection has changed since the iteration began, they throw ConcurrentModificationException. They are implemented by associating a modification count with the collection: if the modification count changes during iteration, `hasNext` or `next` throw ConcurrentModificationException.

An alternative to locking the collection during iteration is to copy the collection and iterate the copy instead.

#### 5.1.3 Hidden iterators

Iteration is also indirectly invoked by the collection's `hashCode` and `equals` methods, which may be called if the collection is used as an element or key of another collection. Similarly, the `containsAll`, `removeAll`, `retainAll` methods, as well as the constructors that take collections as arguments, also iterate the collection.

> Just as encapsulating an object's state makes it easier to preserve the invariants, encapsulating its synchronization make it easier to enforce its synchronization policy.

### 5.2 Concurrent collections

> Replacing synchronized collections with concurrent collections can offer dramatic scalability improvements with little risk.

**ConcurrentHashMap**: a replacement for synchronized hash-based Map implementations.

**CopyOnWriteArrayList**: a replacement for synchronized List implementations.

Queue: A queue is intended to hold elements temporarily while they wait processing.

**ConcurrentLikedQueue**: a traditional FIFO queue.

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

Blocking queues provide blocking *put* and *take* method as well as the timed equivalents *offer* and *poll*. If a queue is full, *put* blocks until space becomes available; if the queue is empty, take blocks until an element is available.

A producer-consumer design separates the  identification of work to be done with the execution of that work by placing work items on a 'to do' list for later processing. BlockingQueue simplifies the *producer-consumer* design pattern.

One of the most common producer-consumer designs is a thread pool coupled with a work queue; this pattern is embodied in the Executor task execution framework.

Blocking queues also provide an *offer* method, which returns a failure status value if the item cannot be enqueued. This enables you create more flexible policies for dealing with overload, such as reducing the number of producer threads.41

> Bounded queues are a powerful resource management tool for building reliable applications: they make you program more robust to overload by throttling activities that threaten to produce more work than can be handled.