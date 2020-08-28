## Java Stream

Optional 类型

`Optional<T>`对象是一种包装器对象，要么包装了类型T的对象，要么没有包装任何对象。对于第一种情况，我们称这种值存为在的。Optional<T>被当作一种更加安全的方式，用来替代类型T的引用，这种引用要么应用某个对象，要么为null。但是只有在正确使用的情况下才会更安全。

如何使用Optional的值

有效地使用Optional值得关键是要使用这样的方法：1、它的值不存在的情况下会产生一个可替代物，2、只有在值存在的情况下才使用这个值。

第一条策略：不存在值的情况下使用某种默认值。

orElse(),orElseGet(),orElseThrow()

第二条策略：只有在存在值的情况下才消费该值。

ifPresent(),map()(看作尺寸0或1的流)

不适合使用Optional值得方式

get, isPresent()

创建Optional得值

Optional.of(result), Optional.empty(), Optional.ofNullable()

用flatMap来构建Optional值的函数

1.8 收集结果

当处理完流之后，通常会想查看其元素。此时可以调用iterator方法，它会产生可以用来访问元素的就是风格的迭代器。或者调用forEach方法将某个函数应用于每个元素。

但跟常见的情况是，我们想要将结果收集到数据结构中。

