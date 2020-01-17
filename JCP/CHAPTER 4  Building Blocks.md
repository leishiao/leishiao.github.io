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

