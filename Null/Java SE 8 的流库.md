## Java SE 8 的流库

**stream**是数据源的一种视图，是一组将被逐个处理的元素。这里的数据源可以是一个数组，Java容器或I/O channel等，Java8支持从这些数据源得到stream。利用stream，我们无需外部迭代数据源中的元素，就可以提取和操作它们。

**流的操作**代表了对stream中每个元素的处理操作。 流的**中间操作**在原stream中嵌入中间操作语义所指定的操作后得到新的stream，可以为新stream继续指定流的操作。这些操作通常组合在一起，在stream上形成一条操作管道，每一个待处理元素都流经这条操作管道，从第一个管道操作开始，前一个管道操作处理元素后将结果传递给下一个管道操作，直到**终结操作**收集每个元素处理的结果。在所有元素都流经管道后，终结操作返回最终处理结果。

流遵循了 ”做什么而非怎么做“ 的原则。函数式接口和流一起使用时自成一体。 函数式编程写出的代码简洁且意图明确，流使得程序更加短小并且更易理解。

> 虽然大部分情况下*stream*是容器调用`Collection.stream()`方法得到的，但*stream*和*collections*有以下不同：
>
> - **无存储**。*stream*不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
> - **为函数式编程而生**。对*stream*的任何修改都不会修改背后的数据源，比如对*stream*执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新*stream*。
> - **惰式执行**。*stream*上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
> - **可消费性**。*stream*只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。
>
> 对*stream*的操作分为为两类，**中间操作(intermediate operations)和结束操作(terminal operations)**，二者特点是：
>
> 1. **中间操作总是会惰式执行**，调用中间操作只会生成一个标记了该操作的新*stream*，仅此而已。
> 2. **结束操作会触发实际计算**，计算发生时会把所有中间操作积攒的操作以*pipeline*的方式执行，这样可以减少迭代次数。计算完成之后*stream*就会失效。

### 流支持

Java 设计者面临着这样一个难题：现存的大量类库不仅为 Java 所用，同时也被应用在整个 Java 生态圈数百万行的代码中。如何将一个全新的流的概念融入到现有类库中呢？ 

比如在 **Random** 中添加更多的方法。只要不改变原有的方法，现有代码就不会受到干扰。 

问题是，接口部分怎么改造呢？特别是涉及集合类接口的部分。如果你想把一个集合转换为流，直接向接口添加新方法会破坏所有老的接口实现类。 

Java 8 采用的解决方案是：在接口中添加 `default`（`默认`）修饰的方法。通过这种方案，设计者们可以将流式（*stream*）方法平滑地嵌入到现有类中。流方法预置的操作几乎已满足了我们平常所有的需求。流操作的类型有三种：创建流，修改流元素（中间操作， Intermediate Operations），消费流元素（终端操作， Terminal Operations）。最后一种类型通常意味着收集流元素（通常是到集合中）。 

### 流类型

stream共有四种类型，分别是`Stream`、`IntStream`、`LongStream`、`DoubleStream`，它们分别处理对象（objects）和基本类型 `int`、`long`、`double`的流。关系如图：

![Package stream](C:\Users\leixiao\Desktop\Package stream.png)



### 创建流

1、**一组元素**转化成为流 。

`Stream.of(T... values))` 
`IntStream.of(int... values)`
`LongStream.of(long... values)`
`DoubleStream.of(double... values)`

```java
Stream<String> stream =Stream.of("It's ", "a ", "wonderful ", "day!");
IntStream intStream = IntStream.of(1, 2, 3, 4, 5, 6);
```

2、**Arrays 类**中含有一个名为 `stream()` 的静态方法用于把任意类型的数组转换成为流。

 对于基本类型`int`, `long`, `double`数组, `stream()`将产生 `IntStream`，`LongStream` 和 `DoubleStream`。 

```java
Stream<String> stringStream = Arrays.stream(new String[]{"hello", "world"});
```

3、每个**集合**都可以通过调用 `stream()` 方法来产生一个流。 

```java
HashSet<String> hashSet = new HashSet<>(Arrays.asList("hello,world.".split(",")))
Stream<String> stream = hashSet.stream();
```

4、**Random** 类被一组生成流的方法增强了。但是Random类只能生成基本类型 `int`，`long`，`double` 的流 。

```java
Random rand = new Random(47);
rand.ints().boxed();//无限流，boxed()流操作将把基本类型包装成为对应的装箱类型
rand.longs(50, 100);// 控制上限和下限：
rand.doubles(2);// 控制流大小
rand.ints(3, 3, 9);// 控制流的大小和界限
```

5、**IntStream** 类提供了 `range()` 方法用于生成整型序列的流。 

```java
public static void repeat(int n, Runnable action)
	{
        range(0, n).forEach(i -> action.run());
    }
```

6、**generate( )**方法可以把任意 `Supplier<T>` 用于生成 `T` 类型的流。 

`Stream.generate(Supplier<T> s)`
`IntStream.generate(IntSupplier s)`
``LongStream.generate(LongSupplier s)``
`DoubleStream.generate(DoubleSupplier s)`

```java
Random rand = new Random(47);
Stream<Integer> integerStream = Stream.generate(rand::nextInt);
IntStream intStream = IntStream.generate(rand::nextInt);
DoubleStream doubleStream = DoubleStream.generate(rand::nextDouble);
```

6、**iterate( )** 根据种子和一元函数迭代生成流。将种子作为一元函数的参数，函数执行的结果作为一个元素加入到流，函数执行的结果同时作为下一次调用函数的参数，得到下一个流元素，以此类推。我们可以利用 `iterate()` 生成一个斐波那契数列。 

`Stream.iterate(final T seed, final UnaryOperator<T> f)`
`IntStream.iterate(final int seed, final IntUnaryOperator f)`
`LongStream.iterate(final long seed, final LongUnaryOperator f)`
`DoubleStream.iterate(final double seed, final DoubleUnaryOperator f)`

```java
int x = 1;//x不是局部变量，应当是类的域
IntStream.iterate(0, i -> {
            int result = x + i;
            x = i;
            return result;
        })
```

7、流的**建造者模式**。 在建造者设计模式（也称构造器模式）中，首先创建一个 `builder` 对象，传递给它多个构造器信息，最后执行“构造”。Stream 库提供了这样的 `Builder`。 

`Stream.builder()`
`IntStream.builder()`
`LongStream.builder()`
`DoubleStream.builder()`

```java
Stream.Builder<String> builder= Stream.builder();
builder.add("Hello");
builder.add("world.");
Stream<String> stringStream = builder.build();
```

9、contact(Stream, Stream)连接两个流，产生新的流。

9、从**Files类**中获取流。 This class consists exclusively of static methods that operate on files, directories, or other types of files. 

10、通过`BufferedReader.lines()`获取流。

### 流操作

#### BaseStream操作

`public interface BaseStream<T, S extends BaseStream<T, S>> extends AutoCloseable`

![1573369293809](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1573369293809.png)

| 操作             | 返回值         | 说明                                                         |
| ---------------- | -------------- | ------------------------------------------------------------ |
| iterator()       | Iterator<T>    | 返回流元素的迭代器                                           |
| spliterator()    | Spliterator<T> | 返回流元素的并行迭代器                                       |
| isParallel()     | boolean        | 流是否会并行执行                                             |
| sequential()     | S              | Returns an equivalent stream that is sequential.             |
| parallel()       | S              | Returns an equivalent stream that is parallel.               |
| unordered()      | S              | Returns an equivalent stream that is unordered.              |
| onClose(Runnale) | S              | Returns an equivalent stream with an additional close handler. |
| close()          |                | 关闭流，执行所有close handler                                |

#### 简单流操作

四种流类型的简单操作，流类型统称为Stream，函数式接口忽略泛型参数，基本类型函数式接口只写通用名称。某些操作基本类型流有而Stream没有，有些则相反。

| 操作                     | 返回值            | 说明                                         |
| ------------------------ | ----------------- | -------------------------------------------- |
| filter(Predicate)        | Stream            | 当前流中满足predicate条件的元素组成的流      |
| map(Function )           | Stream            | 当前流中所有元素执行Function的结果组成的流   |
| mapTo(Function)          | Stream            | 执行Function以转换流的类型                   |
| flatMap(Function)        | Stream            | Function结果是流，摊平这些流                 |
| **flatMapTo(Function)**  | Stream            | Function结果是基本类型流，摊平并转换流的类型 |
| distinct()               | Stream            | 去掉流中的重复元素                           |
| sorted()                 | Stream            | 流中元素按自然顺序排序                       |
| **sorted(Comparator)**   | Stream            | 流中元素按Comparator排序                     |
| asLongStream()           | Stream            | **IntStream转LongStream**                    |
| asDoubleStream()         | Stream            | **LongStream转DoubleStream**                 |
| peek(Consumer)           | Stream            | 调试消费流中元素，不改变流元素               |
| limit(long)              | Stream            | 限制流长度不超过long值                       |
| skip(long)               | Stream            | 丢弃流中前long个元素                         |
| boxed()                  | Stream            | **基本类型流变包装类型**                     |
| forEach(Consumer)        | void              | 使用Consumer消费流中元素                     |
| forEachOrdered(Consumer) | void              | 按顺序消费流中元素                           |
| toArray                  | Object[]          | 返回包含流中元素的数组                       |
| min()                    | Optional          | **基本类型流中最小值**                       |
| **min(Comparator)**      | Optional          | 流中最小值                                   |
| max()                    | Optional          | **基本类型流中最大值**                       |
| **max(Comparator)**      | Optional          | 流中最大值                                   |
| count()                  | long              | 流中元素个数                                 |
| anyMatch(Predicate)      | boolean           | 是否有元素匹配predicate                      |
| allMatch(Predicate)      | boolean           | 是否全部元素匹配predicate                    |
| noneMatch(Predicate)     | boolean           | 是否没油元素匹配predicate                    |
| findFirst()              | Optional          | 返回第一个元素                               |
| findAny()                | Optional          | 返回任意一个元素                             |
| sum()                    | 基本类型          | **基本类型求和**                             |
| average()                | 基本类型          | **基本类型平均值**                           |
| summaryStatistics()      | SummaryStatistics | **获取流元素的各种统计数据**                 |

#### 收集结果

收集结果包括两种方式：reduce和collect

**reduce**：约简，收集数据源元素流经pipeline后的最终结果，可以将所有元素约简为任意类型。

1、将数据约简为最后一个中间操作的输出类型：

```java
T reduce(T identity, BinaryOperator<T> accumulator);
```

identity是流中元素类型的初始中间结果，因此该收集方法不会返回null值。accumulator是一个用于累加的函数，它的第一个参是中间结果，第二个参数是流中的下一个元素，accumulator的返回值作为下一个中间结果，最后一个中间结果作为最终结果。`BinaryOperator<T>`定义如下，由于函数式接口参数的限制，identity类型必须是T类型的值：

```java
public interface BinaryOperator<T> extends BiFunction<T,T,T>{}
```

2、将数据约简为最后一个中间操作的输出类型，初始值默认为null或0：

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
```

由于约简的初始值为null，所以没有元素的流将约简得到值为null的`Optional<T>`,其他与**1**一致。

**3、将数据约简为任意类型：**

```java
<U> U reduce(U identity,
             BiFunction<U, ? super T, U> accumulator,
             BinaryOperator<U> combiner);
```

U是希望约简为的类型，identity作为U类型的**约简初始中间结果**，accumulator定义如何累加中间结果和一个元素，累加结果作为新的中间结果，combiner定义如何合并两个中间结果。

例如，约简字符串流中字符串元素的总长度：

```java
Stream<String> stringStream = Stream.of("It's ", "a ", "wonderful ", "day");
int sumLength = stringStream.reduce(0, (count, s) -> count + s.length(), Integer::sum);
System.out.println(sumLength);//21
```

可以将元素约简到集合中，甚至可以将元素约简到自定义的数据结构中`summaryStatistics`就是例子。

例如，将字符串流中字符串元素约简到字符串和其长度的**映射表**中：

```java
Stream<String> stringStream = Stream.of("It's ", "a ", "wonderful ", "day!");
Map<String, Integer> stringMap = stringStream.reduce(new HashMap<String, Integer>()
                                                     , (map, s) -> {
                                                         map.put(s, s.length());
                                                         return map;
                                                     }
                                                     , (map1, map2) -> {
                                                         map1.putAll(map2);
                                                         return map1;
                                                     });
System.out.println(stringMap);//{It's =5, day!=4, wonderful =10, a =2}
```

reduce作为一种通用约简操作，虽然可以将元素收集到集合中，但是针对收集元素到集合中的操作有专用的操作符collect。

### 可变约简

可变约简操作在处理流中的元素时将输入元素累加到可变结果容器(如`Collection`或`StringBuilder`)中。 

可变的约简操作称为**collection()**，因为它将所需的结果收集到一个结果容器中，例如Collection。一个集合操作需要三个功能：一个供应商函数(supplier)来构造结果容器的新实例，一个累加器函数(accumulator)将一个输入元素合并到一个结果容器中，以及一个组合函数(combiner)将一个结果容器的内容合并到另一个结果容器中。它的形式非常类似于普通简约的一般形式： 

```java
 <R> R collect(Supplier<R> supplier,
               BiConsumer<R, ? super T> accumulator,
               BiConsumer<R, R> combiner);
```

与 reduce() 一样，用这种抽象方式表示约简的好处是它可以直接并行化：只要累积和组合函数满足适当的要求，我们就可以并行地积累部分结果，然后组合它们 。

```java
ArrayList<String> strings = stringStream.collect(() -> new ArrayList<>(),
                                           (c, e) -> c.add(e.toString()),
                                           (c1, c2) -> c1.addAll(c2));
```

收集的三个方面--供应者supplier、累加器accumulator和组合器combiner是紧密耦合的。我们可以使用 收集器Collector接口来得到这三个方面。上面将字符串收集到列表中的示例可以使用标准收集器Collectors重写为： 

```java
List<String> strings = stringStream.collect(Collectors.toList());
```

Collectors中方法较多，详解参见： **[《Java基础系列-Collector和Collectors》](https://www.jianshu.com/p/7eaa0969b424)** 