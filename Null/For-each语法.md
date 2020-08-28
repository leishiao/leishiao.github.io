

# For-each语法

### For-each循环内部实现

For-each语法内部，对collection是用nested iteratoration来实现的，对数组是用下标遍历来实现。

Java 5 及以上的编译器隐藏了基于iteration和下标遍历的内部实现。**在编译的时候编译器会自动将对for这个关键字的使用转化为对目标的迭代器的使用**，**Java将对于数组的foreach循环转换为对于这个数组每一个元素的循环引用**。这就是foreach循环的原理。

进而，我们再得出两个结论：

1、ArrayList之所以能使用foreach循环遍历，是因为ArrayList实现的List接口是Collection的子接口，而Collection是Iterable的子接口，ArrayList的父类AbstractList正确地实现了Iterable接口的iterator方法。

2、任何一个集合，无论是JDK提供的还是自己写的，只要想使用foreach循环遍历，就必须正确地实现Iterable接口

![img](https://upload-images.jianshu.io/upload_images/7687616-4cfcc8518b760f86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/636/format/webp)

### 下面的场合是不适宜使用For-each

使用For-each时对collection或数组中的元素不能做赋值操作

同时只能遍历一个collection或数组，不能同时遍历多个collection或数组

遍历过程中，collection或数组中同时只有一个元素可见，即只有“当前遍历到的元素”可见，而前一个或后一个元素是不可见的。

只能正向遍历，不能反向遍历

