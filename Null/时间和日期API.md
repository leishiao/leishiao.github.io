## 时间和日期API

Java SE8中引入了`java.time` API

时间单位天，小时，分，秒，毫秒，纳秒之间的换算是固定的。但与月，年之间的转换不是固定，一个月不一定是30天，一年也不一定是365天。

### 1、时间线

官方时间：原子钟网络被当作官方时间,偶尔需要插入闰秒。（客观的时间）

Java 使用的时间尺度为：

- 每天86400秒

- 每天正午与官方时间精确匹配

- 在其他时间点上，与官方时间接近匹配

#### Instant

时间线：时间线的原点被称谓“新纪元”（伦敦格林威治皇家天文台，1970年1月1日）。
从该原点开始，时间按照每天86400秒向前或向回度量，精确到纳秒。

Instant表示时间线上的某个**时间点**。

可以将Instant对象用作时间戳。

构建Instant对象的方式有多种，可以与任何表示时间线上时间点的对象相互转换，例如`ZonedDateTime`。

```java
Instant now = Instant.now();
ZonedDateTime zonedDateTime = now.atZone(ZoneId.of("UTC"));
```

#### Duration

Duration是两个时间点之间的**时间量**。

```java
Instant start = Instant.ofEpochSecond(1234567890L);
Instant end= Instant.now();
Duration duration = Duration.between(start, end);
```

#### 运算

Instant和Duration对象的内部存储所需的空间超过了一个`long`的值，因此秒数存储在一个`long`中，而纳秒数存储在一个额外的`int`中。如果想要计算精确到纳秒级，那么实际上需要**秒数**和**纳秒数**两部分内容。

| 描述                                                    | 方法                                   |
| ------------------------------------------------------- | -------------------------------------- |
| 在时间点Instant或时间量Duration上加减**时间量Duration** | `plus`, `minus`                        |
| 在时间点Instant或时间量Duration上加减**给定单位的数值** | 如：`plusNanos`, `minusSeconds`        |
| **缩放**时间量Duration，但不能缩放时间点Instant         | `dividedBy`, `negated`, `multipliedBy` |
| 检查时间量Duration是否是**0或负值**                     | `isZero`, `isNegative`                 |

> Instant 和 Duration 类都是不可修改的类，所有修改实例的计算都会返回一个新的实例。



### 2、本地日期/时间

现在，我们从绝对时间转移到人类时间。在Java API 中有两种人类时间，**本地日期/时间**和**时区时间**。

本地日期/时间包含本地日期`LocalDate`、本地时间`LocalTime`、本地日期时间`LocalDateTime`三个类，与时区信息没有任何关联，不能表示时间线上的时间点，不能直接转换为Instant对象。

Month：表示一年中12个月的枚举类，例如，十月是 `Month.OCTOBER`。

DayOfWeek：表示一周中7天的枚举类，例如，周五是`DayOfWeek.FRIDAY。`

#### LocalDate

LocalDate是带有年、月、日的本地日期。

可以使用now或者of静态方法构建LocalDate对象。

```java
LocalDate localDate = LocalDate.now();
LocalDate programerDay = LocalDate.of(2019, Month.OCTOBER, 24);
```

LocalDate 对象中，年、月、日三个数据作为一个相关联的整体。例如，使用`plusDays`增加天数可能会导致年月数值的变化。

#### Period

Period 存储年、月、日三个量。可以表示两个本地日期LocalDate之间的差值。

period中年、月、日三个量之间没有关系，修改某个量的值不会影响其他量的值。各个量的值大小和正负没有限制。

```java
Period.of(100, -100, 400);//P100Y-100M400D
Period.between(start, end);
```

#### 运算

Period用于基于日历系统的计算，而Duration是基于精确到纳秒的绝对时间量的计算。

例如，可以调用`birthday.plus(Period.ofYears(1))`，来获取下一年的生日，但是一年却不能用Duration表示，因为一年的纳秒数不固定，`birthday.plus(Duration.ofDays(365))`在闰年不会产生正确的结果。

| 描述                                                         | 方法                                            |
| ------------------------------------------------------------ | ----------------------------------------------- |
| 从当前时间或从给定的年月日**构建**LocalDate对象              | `now`, `of`                                     |
| **加减**一定量的**天、星期、月、年**，不能加减小时、分、秒、纳秒 | `plusDays`，`minusWeeks`                        |
| **加减**Duration或Period                                     | `plus`，`minus`                                 |
| **修改**月的日期，年的日期，月、年                           | `withDayOfMonth`，`withDayOfYear`，`withMonth`  |
| **获取**月的日期，年的日期，月、年、星期日期                 | `getDayOfMonth`, `getDayOfYear`, `getDayOfWeek` |
| 获取与另一个日期的**差值**Period                             | `until(endExclusive)`                           |
| 获取与另一个日期指定单位的**差值**，如，差20个月             | `until(endExclusive, ChronoUnit.YEARS))`        |
| 与另一个日期的**前后关系**                                   | `isBefore`，`isAfter`                           |
| 当前年是否是**闰年**                                         | `isLeapYear`                                    |

> 月的日期：day of moth，在1到31之间，当前日期是这个月的第多少天。
>
> 年的日期：day of Year，在1到366之间，当前日期是这一年的第多少天。
>
> 表中的有些方法可能回创建出不存在的日期。例如，在1月31日上加上一个月不应该产生2月31日。这些方法不会抛出异常，而是回返回该月有效的最后一天。
>
> ```java
> LocalDate.of(2019,1,31).plusMonth(1);//会产生2019年2月29日
> ```

除了LocalDate之外，还有MonthDay、YearMonth和Year类可以描述部分日期。例如，12月25日（没有指定年份）可以表示成MonthDay对象。

#### 日期调整器

TemporalAdjusters类提供了大量用于常见调整的静态方法，可以将调整方法传递给with方法。例如：这个月的第一个星期二可以像下面这样计算：

```java
LocalDate.now().with(TemporalAdjusters.nextOrSame(DayOfWeek.THURSDAY));
```

一如既往，with方法会返回一个新的LocalDate对象，而不会修改原来的对象。

TemporalAdjusters类中的日期调整器：

| 描述                                                         | 方法                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 上一个或下一个指定**星期日期**                               | `next(weekday)`，`previous(weekday)`                         |
| **包括当天**的上一个或下一个指定星期日期                     | `nextOrSame(weekday)`，`previousOrSame(weekday)`             |
| 当月中的第n个指定**星期日期**                                | `dayOfWeekInMonth(n, weekday)`                               |
| 当月中的最后一个指定**星期日期**                             | `lastInMonth(weekday)`                                       |
| 当月第一天，当月最后一天，当年最后一天，下月第一天，下一年第一天 | `firstDayOfMonth()`, `lastDayOfMonth()`, `lastDayOfYear()`, `firstDayOfNextMonth(),` `firstDayOfNextYear()` |

除了TemporalAdjusters类中预置的日期调整器，还可以通过实现TemporalAdjuster接口来创建自己的调整器。下面是用于计算下一个工作日的日期调整器。

```java
TemporalAdjuster NEXT_WORKDAY = w ->{
    LocalDate result = (LocalDate)w;
    do {
        result = result.plusDays(1);
    }
    while (result.getDayOfWeek().getValue() >= 6);
    return result;
};

LocalDate backToWork = today.with(NEXT_WORKDAY);
```

注意，对本地日期LocalDate做调整的调整器仅限于调整LocalDate对象，lambda表达式的参数类型为Temporal，他必须被强制转型为 LocalDate，换句话说，该调整器只能用于LocalDate对象。

为了避免这个强制类型转换，可是使用`TemporalAdjusters.ofDateAdjuster()`方法获得一个用于处理LocalDate的TemporalAdjuster。该方法期望得到类型为`UnaryOperator<LocalDate>`的lambda表达式，该lambda表达式引元是LocalDate类型，例如

```java
TemporalAdjuster NEXT_WORKDAY = TemporalAdjusters.ofDateAdjuster(w->{
    LocalDate result = w;
    do {
        result = result.plusDays(1);
    }
    while (result.getDayOfWeek().getValue() >= 6);
    return result;
});
```

#### LocalTime

LocalTime 表示当日本地时间，例如 13:03:00。可以用now或of方法创建其实例：

```java
LocalTime rightNow = LocalTime.now();
LocalTime bedtime = LocalTime.of(22, 30);//22：30：00
```

plus 和 minus 操作是按照一天24小时循环操作的。例如：

```java
LocalTime wakeup = bedtime.plusHours(8);//wakeup is 6：30：00
```

LocalTime 包含**小时，分钟，秒，纳秒**四个量，它们作为一个相互联系的整体，修改一个量的值，其他量的值也可能发生变化。

LocalTime的方法

| 描述                                                       | 方法                           |
| ---------------------------------------------------------- | ------------------------------ |
| 从当前**构建**，或从给定的小时分钟，以及可选的秒和纳秒构建 | `now`, `of`                    |
| **加减**一定量的小时，分钟，秒，纳秒                       | `plusHours`, `minusNamos`      |
| 加减一个精确到纳秒的**Duration**                           | plus, minus                    |
| **修改**小时，分钟，秒，纳秒                               | `withHour`, `withNano`         |
| **获取**小时，分钟，秒，纳秒                               | `getMinute`, `getSecond`       |
| 当天**午夜**到当前LocalTime的秒或纳秒数量                  | `toSecondOfDay`, `toNanoOfDay` |
| 与另一个本地时间的**前后关系**                             | `isBefore`，`isAfter`          |

还有一个表示日期和时间的LocalDateTime类。这个类适合存储固定时区的日期和时间。其内部实现是组合使用了一个LocalDate和一个LocalTime，并将 **年，月，日，小时，分钟，秒，纳秒**作为相互关联的整体，增加或较少某个量，可能会影响其他量的值，例如，加24小时，会导致日期增加一天。该类不考虑时区和夏令时，就如同墙上的挂历和钟表。

如果计算需要处理夏令时，或者需要处理跨时区的用户，那么就应该使用接下来要讨论的ZonedDateTime。

### 3、时区时间

#### ZoneId

Java使用IANA时区数据库。该数据库存储着世界上所有已知时区以及对应的夏令时更变规则。

每个时区都有一个ID，例如 Asia/Shanghai 和 America/Chicago。想要找出当前使用JDK版本所有可用的时区可以调用`ZoneId.getAvailableZoneIds()`。

给定一个时区ID，静态方法`ZondId.of(id)`可以产生一个ZoneId对象。

#### ZoneOffset

ZoneOffset是距离UTC的偏移量，偏移量在`-12:00 ~ +14:00`之间变化，有些时区有小数偏移量，偏移量会随夏令时而发生变化。

#### 夏令时

当夏令时开始时，时钟要向前拨快一小时。反过来，当夏令时结束，时钟要向回拨慢一小时。开始或结束时，时区偏移量会发生变化，在ZonedDateTime中体现为ZoneOffset发生变化。

#### ZonedDateTime

ZonedDateTime 是时间线上一个具体的时间点，可以和Instant完全对应。

任何时区的ZonedDateTime对象都可以转换为Instant对象，Instant对象可以转换为指定时区的ZonedDateTime对象。可以将ZonedDateTime理解为绝对时间Instant在某个时区的视图。

ZonedDateTime 内部实现是使用 LocalDateTime、ZoneOffset、ZoneId三个类的组合实现的，三个量作为一个整体相互联系，同一个Instant时间点，不同的ZoneId或ZoneOffset对应不同LocalDateTime视图。LocalDateTime对象可以通过`local.atZone(zondId)`转换为ZonedDateTime 对象。

```java
Instant now= Instant.now();
//2019-10-27T14:07:33.105+08:00[Asia/Shanghai]
ZonedDateTime shanghai = ZonedDateTime.ofInstant(now,ZoneId.of("Asia/Shanghai"));
//2019-10-27T01:07:33.105-05:00[America/Chicago]
ZonedDateTime chicago = ZonedDateTime.ofInstant(now,ZoneId.of("America/Chicago"));
```

ZonedDateTime的许多方法都与LocalDateTime的方法相同，他们大多数都很直接，但是夏令时带来了一些复杂性。

ZonedDateTime的方法

| 描述                                                         | 方法                                      |
| ------------------------------------------------------------ | ----------------------------------------- |
| 构建ZonedDateTime的方法有多种，关键要能够明确**指定时间点**  | `now`, `of`, `ofInstant`                  |
| 加减**年，月，日，小时，分钟，秒，纳秒，周**，处理了夏令时的问题 | `plusDays`, `minusWeeks`                  |
| 加减一个Duration或Period                                     | `plus`, `minus`                           |
| 修改年，月，月的日期，年的日期，小时，分钟，秒，纳秒         | `withDayOfYear`,`withNano`                |
| 得到给定时区的与当前ZonedDateTime对象同一时刻，或同一本地时间的ZonedDateTime对象 | `withZoneSameInstant`,`withZoneSameLocal` |
| 获取年，月，月的日期，年的日期，小时，分钟，秒，纳秒         | `getDayOfMonth`, `getDayOfWeek`           |
| 获取作为ZoneOffset实例的距离UTC的偏移量。                    | `getOffset`                               |
| 产生本地日期、本地时间、对应的Instant                        | `toLocalDate`, `toLocalTime`, `toInstant` |
| 与另一个ZonedDateTime比较前后关系                            | `isBefore`，`isAfter`                     |

还有一个**OffsetDateTime**类，他表示月UTC具有偏移量的时间，但是没有时区规则的束缚。OffsetDateTime内部实现是使用 LocalDateTime、ZoneOffset两个类的组合实现的，两个量作为一个整体相互联系，同一个Instant时间点，不同的ZoneOffset对应不同LocalDateTime视图。

### 4、格式化和解析

![1572160340234](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572160340234.png)

![1572160543820](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572160543820.png)

![1572160595404](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572160595404.png)

![1572160622001](C:\Users\leixiao\AppData\Roaming\Typora\typora-user-images\1572160622001.png)