# 内部类、lambda表达式

### 内部类

内部类（ inner class）是定义在另一个类中的类。为什么需要使用内部类呢？其主要原因有以下三点：

- 内部类方法可以访问该类定义所在的作用域中的数据，包括私有数据。

- 内部类可以对同一包中的其他类隐藏起来。

- 想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

在Java中有四种内部类：

- 成员内部类

- 静态内部类

- 局部内部类

- 匿名内部类

#### 成员内部类

成员内部类示例：

```java
//外部类
public class OuterClass {
    private SomeType outerField;

    public void outerMethod() {
    }
	//内部类
    public class InnerClass {
        private SomeType innerField;

        public void innerMethod() {
            doSomethingWith(outerField);//访问外部类私有数据域
            doSomethingWith(innerField);//访问自身数据域
        }
    }

    //得到与外部类对象关联的内部类对象，内部类对象访问当前外部类对象数据域
    public InnerClass getInnerClass() {
        return new InnerClass();
    }

    //在外部类方法中使用内部类，内部类对象访问当前外部类对象数据域
    public void useInnerClass() {
        InnerClass innerObject = new InnerClass();
        doSomethingWith(innerObject);
    }
}
```

这里的`InnerClass`位于`OuterClass`类内部，这并不意味着每个外围类实例都有一个内部类实例域。

从传统意义上讲，一个方法可以访问调用这个方法的对象的数据域。内部类方法既可以访问内部类自身对象的数据域，也可以访问创建它的外围类对象的数据域。

内部类的对象总有一个隐式引用，它指向了创建它的外围类对象。外围类的引用在构造器中设置，编译器修改了所有内部类的构造器，添加了一个外围类引用的参数。

外围类的引用在内部类的定义中是不可见的，但是可以在内部类中使用表达式`OuterClass.this`表示外围类的引用。如下代码是合法的，但没有必要。

```java
doSomethingWith(OuterClass.this.outerField);//访问外部类私有数据域
```

内部类编译后类似如下代码：

```java
public class OuterClass$InnerClass {
    private SomeType innerField;
    final OuterClass this$0;
    
    public OuterClass$InnerClass(OuterClass this$0) {
        this.this$0 = this$0;
    }

    public void innerMethod() {
        doSomethingWith(OuterClass.access$000(this.this$0));
        doSomethingWith(this.innerField);
    }
}
```

内部类的构造器必须以外围类对象的引用作为参数。因此，只有创建了外围类对象后才能用外围类创建与该外围类对象关联的内部类对象。

在外围类的作用域之外可以这样引用内部类：`OuterClass.InnerClass`.

创建外部类对象关联的内部类对象通常有三种方式``：

- 外部类提供一个实例方法创建并返回内部类对象，需要的时候在外围类对象上调用该方法获取与之关联的内部类对象，例如

  ```java
  OuterClass.InnerClass innerObject = outerObject.getInnerClass();
  ```

   JDK集合框架中获取集合的迭代器（Iterator）的操作采用了这种方式，从集合中获取的迭代器就是集合的内部类对象。

- 外部类提供一个实例方法创建并使用内部类对象，但不返回对象。例如：`outerObject.useInnerClass()`

- 如果内部类不是声明为private（私有的），那么可以通过外围类对象创建内部类对象，语法为：`outerObject.new InnerClass(construction parameters)`, 例如：

  ```java
  OuterClass.InnerClass  innerObject = outerObeject.new InnerClass();
  ```

如果内部类声明为private（私有的），就只有外围类的方法能够构造内部类对象。只有内部类可以是私有类，而常规类只可以具有包可见性或公有可见性。

内部类中声明的所有静态域都必须是final。内部类不能有static方法。

内部类是一种编译现象，与虚拟机无关。编译器将会把内部类翻译成用$分割外部类名与内部类名的常规类文件，而虚拟机对此一无所知。

正如在上面代码中看到的那样，为了实现在内部类对象中引用外部类对象，编译器修改了内部类的构造方法增加了外部类对象作为参数，并添加了一个this$0域用来保存外部类对象的引用。

事实上，编译器对外部类同样做了修改。为每一个被内部类访问的域，增加了一个以`access$`开头的静态方法，它以外部类对象做为参数，用来返回外部类对象的该域（我们知道，Java中，一个类的方法，不论是实例方法还是类方法，可以访问该类对象的的私有属性）。例如，在上面的例子中，内部类`InnerClass`访问了外围类中的`outerFiled`域，因此编译器会在外围类中增加如下方法：

```java
static boolean access$000(OuterClass outerObject){
    return outerObject.outerFiled;
}
```

编译内部类时，所有对外部类的域的访问，都会被替换为上述静态方法的调用，并用关联的外部类对象作为参数。例如：

```java
doSomethingWith(outerField);//访问外部类私有数据域
```

被替换为：

```java
doSomethingWith(OuterClass.access$000(this.this$0));//通过静态方法访问外围类的域
```

#### 静态内部类

有时候，使用内部类只是为了把一个类隐藏在另一个类的内部，并不需要内部类引用外部类的对象。为此，可以将内部类声明为static，以便取消产生的引用。

只有内部类可以声明为static。静态内部类的对象除了没有对生成它的外围类对象的引用特权外，与其他所有内部类完全一样。

在静态方法中构造的内部类，必须使用静态内部类。

在内部类不需要访问外围类对象的时候，应该使用静态内部类。

与常规内部类不同，静态内部类可以有静态域和方法。

声明在接口中的内部类自动成为static和public类。

#### 局部内部类

可以在一个方法中定义局部类。

```java
public void start(int interval,boolean beep){
    class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("At the tone,the tine is " + new Date());
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
    ActionListener actionListener = new TimePrinter();
    Timer timer = new Timer(interval, actionListener);
    timer.start();
}
```

局部类不能用public或private访问说明符进行声明。他的作用域被限定在声明在这个局部类的块中。

与其他内部类相比较，局部类还有一个优点，它们不仅能够访问包含他们的外部类，还可以访问局部变量。不过，那些局部变量必须事实上为final。这说明，它们一旦赋值就绝不会改变。

局部内部类使用方法的局部变量，当方法结束，局部变量将不复存在。为了能让内部类在外部方法结束后还能使用方法的局部变量，需要在方法结束前将方法的局部变量备份到新创建的局部内部类对象中。

局部内部类的做法是，在局部变量的构造方法中，除了传入对外围类对象的引用外，还要传入局部变量，并将这些变量保存在创建的内部类对象对应的域中。创建局部内部类对象的时候，局部变量将被传递给构造器，并保存在内部类中。

编译器必须检测对局部变量的访问，为每一个局部变量建立相应的数据域，并将局部变量拷贝到构造其中，以便将这些数据域初始化为局部变量的副本。

上诉内部类代码编译后类似：

```java
class OuterCalss$TimePrinter implements ActionListener {
    final boolean var$beep;
    final OuterCalss this$0
    OuterCalss$TimePrinter(OuterCalss this$0, boolean var2) {
        this.this$0 = this$0;
        this.val$beep = var2;
    }

    public void actionPerformed(ActionEvent e) {
        System.out.println("At the tone,the tine is " + new Date());
        if (this.val$beep) {
            Toolkit.getDefaultToolkit().beep();
        }
    }
}
```

前面曾经提到，局部类的方法只可以引用定义为final的局部变量，Java 8 会自动把从局部类访问的局部变量声明为final。

有时，final限制显得并不太方便。可以通过数组或者final的对象引用，然后在局部类中修改数组的值，或者调用对象引用的方法修改对象状态。例如：

```java
public void method(){
	final AtomicInteger atomicInteger = new AtomicInteger();
    class InnerClass{
        public void innerMethod(){
            atomicInteger.incrementAndGet();
        }
    }
}
```

#### 匿名内部类

将局部内部类的使用再深入一步。假如只创建这个类的一个对象，就不必命名了。这个类被称为匿名内部类。

将上面的代码用匿名内部类实现：

```java
public void start(int interval, boolean beep) {
    ActionListener listener = new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("At the tone,the tine is " + new Date());
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    };

    Timer timer = new Timer(interval, listener);
    timer.start();
}
```

这种语法确实有些难以理解。它的涵义是：创建一个实现ActionListener接口的新类的新对象，需要实现的方法  `actionPerformed`定义在{}内。

通常的语法格式为：

```
new SuperType(construction parameters)
{
    inner class methods and data 
}
```

其中，`SuperType` 可以是一ActionListener这样的接口，于是内部类就要实现这个接口。`SuperType` 也可以是一个类，于是内部类就要扩展它。

由于构造器的名字必须与类名相同，而匿名类没有类名，所以，匿名不能有构造器。取而代之的是，将构造器参数传递给**超类**构造器。尤其是在内部类实现接口的时候，不能有任何构造器参数。不仅如此，还有像下面这要提供一组括号：

```
new InterfaceType()
{
    methods and data 
}
```

与局部内部类相比，匿名内部类的解决方案比较简短、更切实际、更易于理解。

多年来，Java 程序员习惯的做法是用匿名内部类实现事件监听器和其他回调。如今，最好还是使用**lambda**表达式。



### lambda 表达式

你会了解如何使用 lambda 表达式采用一种简洁的语法**定义代码块**，如何编写处理lambda 表达式的代码。

lambda表达式是**函数式接口**的实现。

#### 为什么引入 lambda 表达式

lambda 表达式是一个可存储的可传递的代码块 ， 可以在以后执行一次或多次。本质上就是一个函数，需要输入参数，需要返回输出结果。

Java SE8之前， 在 Java 中传递一个代码段并不容易 ，不能直接传递代码段。 Java 是一种面向对象语言，所以必须构造一个对象 ， 这个对象的类需要有一个方法能包含所需的代码。

在其他语言中，可以直接处理代码块，它们的大部分 API都更简单 、更一致而且更强大。在 Java 中 ，也可以编写类似的 API 利用类对象实现特定的功能，不过这种 API 使用可能很不方便。

就现在来说，问题已经不是是否增强 Java 来支持函数式编程 ，而是要如何做到这一点 。

#### lambda 表达式的语法

Java 中的 lambda 表达式形式是 : **参数 ， 箭头 （ - > ) 以及一个表达式**。

在这种lambda形式中，需要输入的是**参数**，返回的是**表达式**的结果。每次调用lambda表达式需要输入参数，参数作为lambda输入值，可以参与表达式的运算。

如果代码要完成的计算无法放在一个表达式中，就可以像写方法一样 ， 把这些代码放在{}中 ，并包含**显式**的 return 语句。下面是符合形式的两个lambda表达式:

```java
//输输入是两个字符串，输出是表达式结果
(String first, String second)->first.length() - second.length();
//输出是{}中代码块显示return的值
(String first ,String second)->{
	if (first.length() < second.length()) 
        return -1 ;
	else if (first.length() > second.length())
        return 1 ;
	else return 0 ;
}
```

#### 函数式接口

Java 中已经有很多封装代码块的接口， 如 ActionListener 或Comparator 。

根据以往的经验，用带单个抽象方法的接口作为**函数类型**。他们的实例称为**函数对象**，表示函数或者要采取的动作。在Java8之前，创建函数对象的主要方式是通过匿名内部类。要注意，匿名内部类对象不一定都是函数对象。

在Java 8中，形成了”带有单个抽象方法的结构是特殊的，值得特殊对待“的观念，这些只有一个抽象方法的接口现在称为函数式接口。

Java允许利用lambda表达式创建这些接口的实例。需要这种接口的实例的地方，就可以提供一个 lambda 表达式。

在Java 中，对lambda 表达式所能做的也只是能转换为函数式接口的实例。

#### 用lambda实现函数式接口

用lambda实现函数式接口的根本在于遵循函数是接口的约定，实现接口便是承诺履行接口的所有约定。不遵守约定，必然得不到预期的结果。

在Java中，lambda表达式必须符合某个函数式接口的定义，即符合函数式接口中唯一抽象方法的定义，不符合函数式接口的lambda表达式既不能作为变量的值存储，也不能作为方法调用的参数。具体来说：

1、函数式接口中方法的**参数个数和参数类型**是确定的，实现该接口的lambda表达式必须在**参数**部分声明相同个数和类型的参数。

2、函数式接口中方法的**返回类型**是确定的，实现该接口的lambda表达式必须返回唯一抽象方法声明返回类型的值。

符合以上两点的lambda表达式，就可以存储在对应函数式接口的变量中，也可以当作该接口的一个实例，作为方法调用的参数。

例如，Comparator接口是一个函数式接口，定义如下：

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

下名是遵循`Comparator<String>`接口定义的一个lambda表达式：

```java
Comparator<String> comparator = (String first,String second) ->{return first.length() -second.length();};
```

这个lambda表达式接受两个`String`类型的参数，返回一个`int`值，在语法上是完全符合函数式接口`Comparator<String>`的定义，因此可以保存在`Comparator<String>`类型的变量comparator中。

如果对函数式接口足够熟悉，可以不写明参数类型，和return语句，只有一个参数时还可以去掉参数括号：

```java
Comparator<String> comparator = (first,second) ->first.length() -second.length();
//实现该lambda的程序员要明白，first和second是String类型，也只能用作String类型，还要知道反回类型是int
```

要正确使用lambda表达式实现一个函数式接口，需要对该接口抽象方法的定义了然于心，包括参数列表，返回值类型约定。lambda表达式不仅要语法正确，更要功能正确。

Java API 在`java.util.function`包中定义了很多非常通用的函数式接口。

其中一个接口  `BiFunction<T,U,R>` 描述了参数类型为T和U而返回类型为R的函数。我们可以把我们的字符串比较 lambda 表达式保存在这个类型的变量中：

```
BiFunction<String,String,Integer> comp = (first,second)->first.length()-second.length();
```

`java.util.function`包中有一个尤其有用的接口`Predicate`：

```java
public interface Predicate<T> {
    boolean test(T t);
}

```

`ArrayList`类有一个`removeIf`方法，它的参数就是一个Predicate。这个接口专门用来传递lambda表达式。例如，下面的语句将从一个数组列表删除所有的null值：

```java
list.removeIf(e-> e==null);
```

#### 方法引用

如果lambda表达式的代码块**只用**传递给lambda表达式的参数调用一个方法，那么就可以用**方法引用**代替lambda表达式。

例如：假设你希望只要出现一个定时器事件就打印这个事件对象。可以使用如下lambda调用：

```java
Timer timer = new Timer(1000, e -> System.out.println(e));//lambda方式
```

但是，因为lambda表达式只用参数执行的一个方法调用，因此也可以使用方法引用：

```java
Timer timer = new Timer(1000, System.out::println);//方法引用方式
```

表达式 `System.out::println`是一个方法引用（method reference），它等价于 lambda表达式
` x -> System.out.println(x)`

方法引用要用 `::`操作符分割方法名与对象名或类名。主要有三种情况

- `object::instanceMethod`

- `Class::staticMethod`

- `Class::instanceMethod`

前两种情况中，方法引用等价于提供方法参数的 lambda表达式。

第一种情况，前面已经提到，`System.out::println`等价于` x -> System.out.println(x)`。

第二种情况， `Math::pow`等价于 `(x,y)->Math.pow(x,y)`。

第三种情况，第一个参数会成为方法的目标。 `String::equals`等价于`(x,y)->x.equals(y)`。因此，``Class::instanceMethod``中的Class必须是第一个参数的类型。

可以在方法引用中使用this参数。例如，`this.equals`等同于 `x-this.equals(x)`。

使用super也是合法的。下面的表达式：`super::instantceMethod` 使用this作为目标，会调用给定方法的超类版本。可以理解为等价于 `(x,y)->this.super_instantceMethod(x,y)` ，这里 `super_instantceMethod`是 `instantceMethod的超类版本`。

类似于lambda表达式，方法引用不能独立存在，总是会转换为函数式接口的实例。

要理解方法引用，可以把方法引用转换为lambda表达式，lambda表达式更容易理解。如果存在多个重载的方法，需要清楚的知道**方法引用实现的函数式接口**，才能将其转换为正确的lambda表达式。

方法引用与lambda表达式的转换：

| 方法引用                 | lambda表达式                          |
| ------------------------ | ------------------------------------- |
| `object::instanceMethod` | `x -> object.instanceMethod(x)`       |
|                          | `(x,y) -> object.instanceMethod(x,y)` |
| `Class::staticMethod`    | `x -> Class.staticMethod(x)`          |
|                          | `(x,y) -> Class.staticMethod(x,y)`    |
| `Class::instanceMethod`  | `x  -> x.instanceMethod()`            |
|                          | `(x,y)  -> x.instanceMethod(y)`       |

#### 构造器引用

构造器引用与方法引用类似，只不过方法名为 new。

例如 `ClassName::new` 是 `ClassName`类的构造器的一个引用。

也可以用数组类型建立构造器引用。

例如，`int[]::new`是一个构造器引用，他有一个参数：即数组的长度。这等价于lambda表达式 `x->new int[x]`

构造器引用与lambda表达式的转换：

| 构造器引用    | lambda表达式               |
| ------------- | -------------------------- |
| `Class::new`  | `x -> new Class(x)`        |
|               | `(x, y) -> new Class(x,y)` |
| `Type[]::new` | `x -> new Type[x]`         |

要将构造器引用转换为lambda表达式，同样需要了解构造器引用**要实现的函数式接口**。

无论是方法引用还是构造器引用，含义都不太直观，想要正确使用都比较复杂。

我们最好根据要实现的函数式接口，写容易理解的lambda表达式，然后再优化为等价的方法引用或者构造器引用。

而理解代码中的方法引用或构造器引用时，可以根据其要实现的函数式接口，将其转换为等价的更加直观的lambda表达式。

由此可见，了解要实现的函数式接口和它的约定，是正确使用lambda，方法引用，构造器引用的基础。

#### 变量作用域

通常，我们会在lambda表达式中访问外围方法或外围类中的变量。但是，当lambda表达式被执行的时候，这些外围方法的变量可能已经不存在了。如何保留这些局部变量的值呢？

要了解到底会发生什么，下面来巩固我们对lambda表达式的理解。

lambda表达式有三个部分：

1. 参数
2. 一个代码块
3. 自由变量的值，这是指**非参数**而且**不在代码中定义**的变量，即所访问的外围方法或类的变量。

表示lambda的数据结构必须存储这些自由变量的值，我们说这些自由变量被lambda表达式捕获。（如何捕获？例如，可以把lambda表达式转换为包含一个方法的对象，这样自由变量的值就会复制到这个对象的实例变量中。）

可以看到，lambda表达式可以捕获外围作用域中的变量的值。在java中，要确保所捕获的值是明确定义的，这里有一个重要原则。lambda表达式中，只能引用值不会改变的变量。

之所以有这个限制是有原因的。如果在lambda表达式中改变变量，并发执行多个动作时就会不安全。另外如果在lambda表达式中引用变量，而这个变量可能在外部被改变，这也是不合法的。

这里有一条规则：lambda表达式中捕获的变量必须是事实上的最终变量。事实上的最终变量是指，这个变量初始化之后就不会再为它赋新值。

lambda表达式的体与嵌套块有相同的作用域，这里同样适用命名冲突和遮蔽的有关规则。在lambda表达式中声明与局部变量同名的参数或变量是不合法的。

在一个lambda表达式中使用this关键字时，是指创建这个lambda表达式的方法的this参数。（只有在方法中才能使用this，是指调用方法时的隐式参数。）

#### 处理lambda表达式

到目前为止，已经了解了如何生成lambda表达式，以及如何把lambda表达式传递到需要一个函数式接口的方法。下面来看如何编写处理lambda表达式的方法。

使用lambda表达式的重点是延迟执行。毕竟，如果想要立即执行代码，完全可以直接执行，而无需把它包装在一个lambda表达式中。之所以希望以后执行代码，这有很多原因，如：

- 在一个单独的线程中运行代码
- 多次运行代码
- 在算法的适当位置运行代码（排序中的比较操作）
- 发生某种情况时执行代码（点击事件，数据到达）
- 只在必要时才运行代码

下面来看一个简单的例子。假设想要执行一个动作n次。将这个动作和重复次数传递到一个repeat方法：

```java
repeat(10,()->System.out.printlin("Hello,world"));
```

要接受这个lambda表达式，需要选择（有时候需要编写）一个函数式接口。这里可以使用Runnable接口:

```java
public static void repeat(int n,Runnale action){
	for(int i=0;i<n;i++) action.run();
}
```

调用`action.run()`时会执行这个lambda表达式的主体。

所有的函数式接口都有一个特定名称的抽象方法，通过调用该抽象方法并提供参数，来执行实现了该接口的函数对象。

Java提供了大量通用的函数式接口，这些接口可以在 `java.util.function`包下找到。

![1572783953440](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572783953440.png)

![1572783893354](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572783893354.png)