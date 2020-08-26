# Interfaces

*Core collection interfaces* 包含不同类型的集合接口，如图所示。这些集合接口可以用来操作集合，而与集合的具体实现无关。*Core collection interfaces* 是Java Collections Framework的基础。如你所见，*core collection interfaces* 形成了层次结构。

![Two interface trees, one starting with Collection and including Set, SortedSet, List, and Queue, and the other starting with Map and including SortedMap.](https://docs.oracle.com/javase/tutorial/figures/collections/colls-coreInterfaces.gif)

Set 是一种特殊的 Collection，SortedSet 是一特殊的 Set, 以此类推。要注意到，这个层次结构包括两个不同的树形结构—— Map 不是 Collection.

注意，*core collection interfaces* 都是泛型，比如，Collection 接口定义如下：

```java
public interface Collection<E>...
```

`<E>` 说明这个接口是泛型。当声明一个 Collection 变量时，你应该指定集合所含元素的类型。指定类型让编译器能够检查（编译期）放进集合的对象类型是正确的，这样减少了运行时错误。关于泛型的资料，参考泛型相关课程。

理解了怎样使用这些接口就知道了大部分关于 Java Collections Framework 的内容。本章讨论有效使用这些接口的一般原则，包括什么时候用哪个接口。你也会学到使用每种接口的编程习语（programming idioms），以便充分利用该接口。

为了让 core collection interfaces 容易使用，Java没有为集合接口的变体提供单独的接口，这些变体包括不可变集合（immutable），固定大小集合（ fixed-size），仅追加集合（append-only）。为此，在集合接口中的修改操作，都被设计为可选的（*optional* ）—— 某些接口实现类可以选择不实现所有的操作。如果调用一个未实现的操作，会抛出`UnsupportedOperationException`异常。集合接口实现类有责任在文档中指出支持哪些操作。Java中所有通用实现类（general-purpose implementations）都支持全部可选操作。

如下列表描述了core collection interfaces：

- `Collection` — the root of the collection hierarchy. 集合层次结构的根。一个集合代表一组对象，这些对象被称为该集合的元素（*elements*）。Collection 接口是最一般的接口，所有集合都实现了它，并且在需要最大通用性（maximum generality）时，被用来传递和操作集合对象。一些集合类型允许重复的元素，其他不允许。一些是有序的，其他是无序的。Java 不提供这个接口的直接实现类，但是提供了更特殊化的子接口的实现类，比如 Set 和 List 接口。

- `Set` — a collection that cannot contain duplicate elements. 不能包含重复元素的集合。该接口是对数学中的集进行建模，并用于表示集，如扑克牌、组成学生日程的课程或在机器上运行的进程等。

- `List` — an ordered collection (sometimes called a *sequence*).  一个有序的集合。列表（`List`s ）可以包含重复的元素。一个List的用户能够准确控制每个元素插入列表的位置，并使用整数索引（position）访问元素。如果使用过 Vector, 你就熟悉List的一般风格。

- `Queue` — a collection used to hold multiple elements prior to processing. 队列，存放多个待处理元素的集合。除了基本的 Collection 的操作，Queue还提供额外的插入、提取和检查操作。

  队列通常使用FIFO (first-in, first-out) 方式对元素排序，但不是必须的。优先级队列（priority queues）就是例外，它根据一个外部提供的比较器（comparator）或者元素的自然顺序（natural ordering）对元素排序。不论使用什么顺序，队列头是 remove 或 poll 方法要删除的元素。在FIFO队列中，所有新元素被插入到队列末尾。其他类型的队列可以使用不同的放置规则。每一个Queue的实现类都必须指定其排序特征（ordering properties）。

- `Deque` — a collection used to hold multiple elements prior to processing. 双端队列，存放多个待处理元素的集合。除了基本的 Collection 的操作，Queue还提供额外的插入、提取和检查操作。

  Deques既可用作FIFO(first-in, first-out)，也可用作LIFO(last-in, first-out).在双端队列中，所有新元素都可以在两端插入、检索和删除。

- `Map` — an object that maps keys to values. 将键（key）映射值（value）的对象。Map不能包含重复的键，每个键最多只能映射到一个值。如果使用过 Hashtable，那么您已经熟悉Map的基础知识了。

最后两个 core collection interfaces 仅仅是 Set 和 Map的排序版本（sorted versions）:

- `SortedSet` — a `Set` that maintains its elements in ascending order. 有序集，一个按照升序管理（maintains ）元素的Set。额外提供了几个利用其顺序特征的的操作。排序集用于具有自然顺序的集（naturally ordered sets），例如单词列表，或花名册（membership rolls）。
- `SortedMap` — a `Map` that maintains its mappings in ascending key order. 一个按照建（key）升序管理（maintains）映射关系（mappings）的 Map。This is the `Map` analog of `SortedSet`。有序的映射用于具有自然顺序的（naturally ordered）键值对集合，例如字典和电话簿。