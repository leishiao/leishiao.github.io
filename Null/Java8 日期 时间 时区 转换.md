# Java8 日期 时间 时区 转换 

1、当前时间：距离标准时间（1970年01月01日 00:00:00）的毫秒值。

2、java提供java.time包，用于时间的处理。

一、简介

　　在Java8之前，日期时间API一直被开发者诟病，包括：java.util.Date是可变类型，SimpleDateFormat非线程安全等问题。故此，Java8引入了一套全新的日期时间处理API，新的API基于ISO标准日历系统。

　　　　

二、日期初识

　　示例1： 获取当天日期

　　　　Java 8中的 LocalDate 用于表示当天日期。和java.util.Date不同，它只有日期，不包含时间。

```java
public static void main(String[] args) {
　　LocalDate date = LocalDate.now();
　　System.out.println("当前日期=" + date);
}
```

 

　　示例2： 构造指定日期

　　　　调用工厂方法LocalDate.of()创建任意日期， 该方法需要传入年、月、日做参数，返回对应的LocalDate实例。这个方法的好处是没再犯老API的设计错误，比如年度起始于1900，月份是从0开 始等等

```java
public static void main(String[] args) {
    LocalDate date = LocalDate.of(2000, 1, 1);
    System.out.println("千禧年=" + date);
}
```

 

　　示例3： 获取年月日信息

```java
public static void main(String[] args) {
    LocalDate date = LocalDate.now();
    System.out.printf("年=%d， 月=%d， 日=%d"
                      , date.getYear(), date.getMonthValue(), date.getDayOfMonth());
}
```

 

　　示例4： 比较两个日期是否相等

```java
public static void main(String[] args) {
    LocalDate now = LocalDate.now();
    LocalDate date = LocalDate.of(2018, 9, 24);
    System.out.println("日期是否相等=" + now.equals(date));
}
```

 

三、时间初识

　　示例： 获取当前时间

　　　　Java 8中的 LocalTime 用于表示当天时间。和java.util.Date不同，它只有时间，不包含日期。

```java
public static void main(String[] args) {
    LocalTime time = LocalTime.now();
    System.out.println("当前时间=" + time);
}
```

 

四、比较与计算

　　示例1： 日期时间计算

　　　　Java8提供了新的plusXxx()方法用于计算日期时间增量值，替代了原来的add()方法。新的API将返回一个全新的日期时间示例，需要使用新的对象进行接收。

```
    public static void main(String[] args) {
        
　　　　 // 时间增量
        LocalTime time = LocalTime.now();
        LocalTime newTime = time.plusHours(2);
        System.out.println("newTime=" + newTime);
        
　　　　　// 日期增量
        LocalDate date = LocalDate.now();
        LocalDate newDate = date.plus(1, ChronoUnit.WEEKS);
        System.out.println("newDate=" + newDate);
        
    }
```

 

　　示例2： 日期时间比较

　　　　Java8提供了isAfter()、isBefore()用于判断当前日期时间和指定日期时间的比较

```java
    public static void main(String[] args) {
        
        LocalDate now = LocalDate.now();
        
        LocalDate date1 = LocalDate.of(2000, 1, 1);
        if (now.isAfter(date1)) {
            System.out.println("千禧年已经过去了");
        }
        
        LocalDate date2 = LocalDate.of(2020, 1, 1);
        if (now.isBefore(date2)) {
            System.out.println("2020年还未到来");
        }
        
    }
```

 

五、时区

　　示例： 创建带有时区的日期时间

　　　　Java 8不仅分离了日期和时间，也把时区分离出来了。现在有一系列单独的类如ZoneId来处理特定时区，ZoneDateTime类来表示某时区下的时间。

```java
    public static void main(String[] args) {
        
        // 上海时间
        ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
        ZonedDateTime shanghaiZonedDateTime = ZonedDateTime.now(shanghaiZoneId);
        
        // 东京时间
        ZoneId tokyoZoneId = ZoneId.of("Asia/Tokyo");
        ZonedDateTime tokyoZonedDateTime = ZonedDateTime.now(tokyoZoneId);
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        System.out.println("上海时间: " + shanghaiZonedDateTime.format(formatter));
        System.out.println("东京时间: " + tokyoZonedDateTime.format(formatter));
        
    }
```

 

六、格式化

　　示例1: 使用预定义格式解析与格式化日期

```java
　　public static void main(String[] args) {
        
        // 解析日期
        String dateText = "20180924";
        LocalDate date = LocalDate.parse(dateText, DateTimeFormatter.BASIC_ISO_DATE);
        System.out.println("格式化之后的日期=" + date);
        
        // 格式化日期
        dateText = date.format(DateTimeFormatter.ISO_DATE);
        System.out.println("dateText=" + dateText);
        
    }
```

 

　　示例2： 日期和字符串的相互转换

```java
    public static void main(String[] args) {
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        // 日期时间转字符串
        LocalDateTime now = LocalDateTime.now();
        String nowText = now.format(formatter);
        System.out.println("nowText=" + nowText);
        
        // 字符串转日期时间
        String datetimeText = "1999-12-31 23:59:59";
        LocalDateTime datetime = LocalDateTime.parse(datetimeText, formatter);
        System.out.println(datetime);
        
    }
```

 

七、相关类说明

```
Instant         时间戳
Duration        持续时间、时间差
LocalDate       只包含日期，比如：2018-09-24
LocalTime       只包含时间，比如：10:32:10
LocalDateTime   包含日期和时间，比如：2018-09-24 10:32:10
Peroid          时间段
ZoneOffset      时区偏移量，比如：+8:00
ZonedDateTime   带时区的日期时间
Clock           时钟，可用于获取当前时间戳
java.time.format.DateTimeFormatter      时间格式化类
```



八、时间戳、时区关系

​		时间戳和时区没有关系，所有时区的时间戳是一样的。可以根据时间戳计算出指定时区的时间表示，也可以根据时区和时间表示计算出该时区时间对应的时间戳。