---
title: Java8新的日期API LocalDate, LocalTime
date: 2019-04-20 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

由于Java Date的各种问题，Java8推出了新的日期API，很受一拨人的追捧。

<!--more-->

#### 为什么我们需要新的Java日期/时间API？

在开始研究Java 8日期/时间API之前，让我们先来看一下为什么我们需要这样一个新的API。在Java中，现有的与日期和时间相关的类存在诸多问题，其中有：

- Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。
- java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
  对于时间、时间戳、格式化以及解析，并没有一些明确定义的类。对于格式化和解析的需求，我们有java.text.DateFormat抽象类，但通常情况下，SimpleDateFormat类被用于此类需求。
- 所有的日期类都是可变的，因此他们都不是线程安全的，这是Java日期类最大的问题之一。
- 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

在现有的日期和日历类中定义的方法还存在一些其他的问题，但以上问题已经很清晰地表明：Java需要一个健壮的日期/时间类。这也是为什么Joda Time在Java日期/时间需求中扮演了高质量替换的重要角色。

#### Java 8日期/时间API

Java 8日期/时间API是JSR-310的实现，它的实现目标是克服旧的日期时间实现中所有的缺陷，新的日期/时间API的一些设计原则是：

- 不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程环境有好处。
- 关注点分离：新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
- 清晰：在所有的类中，方法都被明确定义用以完成相同的行为。举个例子，要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
- 实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
- 可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。

#### Java日期/时间API包

Java日期/时间API包含以下相应的包。

1. `java.time`包：这是新的Java日期/时间API的基础包，所有的主要基础类都是这个包的一部分，如：LocalDate, LocalTime, LocalDateTime, Instant, Period, Duration等等。所有这些类都是不可变的和线程安全的，在绝大多数情况下，这些类能够有效地处理一些公共的需求。
2. `java.time.chrono`包：这个包为非ISO的日历系统定义了一些泛化的API，我们可以扩展AbstractChronology类来创建自己的日历系统。
   java.time.format包：这个包包含能够格式化和解析日期时间对象的类，在绝大多数情况下，我们不应该直接使用它们，因为java.time包中相应的类已经提供了格式化和解析的方法。
3. `java.time.temporal`包：这个包包含一些时态对象，我们可以用其找出关于日期/时间对象的某个特定日期或时间，比如说，可以找到某月的第一天或最后一天。你可以非常容易地认出这些方法，因为它们都具有“withXXX”的格式。
4. `java.time.zone`包：这个包包含支持不同时区以及相关规则的类。

#### Java日期/时间API示例

我们已经浏览了Java日期/时间API的大多数重要部分，现在是时候根据示例仔细看一下最重要的一些类了。

1. java.time.LocalDate：LocalDate是一个不可变的类，它表示默认格式(yyyy-MM-dd)的日期，我们可以使用now()方法得到当前时间，也可以提供输入年份、月份和日期的输入参数来创建一个LocalDate实例。该类为now()方法提供了重载方法，我们可以传入ZoneId来获得指定时区的日期。该类提供与java.sql.Date相同的功能，对于如何使用该类，我们来看一个简单的例子。

   ```java
   @Test
   public void testLocalDate() {
       //Current Date
       LocalDate today = LocalDate.now();
       System.out.println("Current Date=" + today);
   
       //Creating LocalDate by providing input arguments
       LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
       System.out.println("Specific Date=" + firstDay_2014);
   
       //Try creating date by providing invalid inputs
       //LocalDate feb29_2014 = LocalDate.of(2014, Month.FEBRUARY, 29);
       //Exception in thread "main" java.time.DateTimeException:
       //Invalid date 'February 29' as '2014' is not a leap year
   
       //Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
       LocalDate todayKolkata = LocalDate.now(ZoneId.of("Asia/Kolkata"));
       System.out.println("Current Date in IST=" + todayKolkata);
   
       //java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
       //LocalDate todayIST = LocalDate.now(ZoneId.of("IST"));
   
       //Getting date from the base date i.e 01/01/1970
       LocalDate dateFromBase = LocalDate.ofEpochDay(365);
       System.out.println("365th day from base date= " + dateFromBase);
   
       LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
       System.out.println("100th day of 2014=" + hundredDay2014);
   }
   ```

   打印

   ```
   Current Date=2018-05-29
   Specific Date=2014-01-01
   Current Date in IST=2018-05-29
   365th day from base date= 1971-01-01
   100th day of 2014=2014-04-10
   ```

2. java.time.LocalTime：LocalTime是一个不可变的类，它的实例代表一个符合人类可读格式的时间，默认格式是hh:mm:ss.zzz。像LocalDate一样，该类也提供了时区支持，同时也可以传入小时、分钟和秒等输入参数创建实例，我们来看一个简单的程序，演示该类的使用方法。

   ```java
   @Test
   public void testLocalTime() {
       //Current Time
       LocalTime time = LocalTime.now();
       System.out.println("Current Time=" + time);
   
       //Creating LocalTime by providing input arguments
       LocalTime specificTime = LocalTime.of(12, 20, 25, 40);
       System.out.println("Specific Time of Day=" + specificTime);
   
       //Try creating time by providing invalid inputs
       //LocalTime invalidTime = LocalTime.of(25,20);
       //Exception in thread "main" java.time.DateTimeException:
       //Invalid value for HourOfDay (valid values 0 - 23): 25
   
       //Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
       LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));
       System.out.println("Current Time in IST=" + timeKolkata);
   
       //java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
       //LocalTime todayIST = LocalTime.now(ZoneId.of("IST"));
   
       //Getting date from the base date i.e 01/01/1970
       LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
       System.out.println("10000th second time= " + specificSecondTime);
   }
   ```

   打印

   ```
   Current Time=19:09:39.656
   Specific Time of Day=12:20:25.000000040
   Current Time in IST=16:39:39.657
   10000th second time= 02:46:40
   ```

3. java.time.LocalDateTime：LocalDateTime是一个不可变的日期-时间对象，它表示一组日期-时间，默认格式是yyyy-MM-dd-HH-mm-ss.zzz。它提供了一个工厂方法，接收LocalDate和LocalTime输入参数，创建LocalDateTime实例。我们来看一个简单的例子。

   ```java
   @Test
   public void testLocalDateTime() {
       //Current Date
       LocalDateTime today = LocalDateTime.now();
       System.out.println("Current DateTime=" + today);
   
       //Current Date using LocalDate and LocalTime
       today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
       System.out.println("Current DateTime=" + today);
   
       //Creating LocalDateTime by providing input arguments
       LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
       System.out.println("Specific Date=" + specificDate);
   
       //Try creating date by providing invalid inputs
       //LocalDateTime feb29_2014 = LocalDateTime.of(2014, Month.FEBRUARY, 28, 25,1,1);
       //Exception in thread "main" java.time.DateTimeException:
       //Invalid value for HourOfDay (valid values 0 - 23): 25
   
       //Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
       LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
       System.out.println("Current Date in IST=" + todayKolkata);
   
       //java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
       //LocalDateTime todayIST = LocalDateTime.now(ZoneId.of("IST"));
   
       //Getting date from the base date i.e 01/01/1970
       LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
       System.out.println("10000th second time from 01/01/1970= " + dateFromBase);
   }
   ```

   在所有这三个例子中，我们已经看到如果我们提供了无效的参数去创建日期/时间，那么系统会抛出java.time.DateTimeException，这是一种运行时异常，我们并不需要显式地捕获它。

   同时我们也看到，能够通过传入ZoneId得到日期/时间数据，你可以从它的Javadoc中得到支持的Zoneid的列表，当运行以上类时，可以得到以下输出

   打印

   ```
   Current DateTime=2018-05-29T19:10:00.353
   Current DateTime=2018-05-29T19:10:00.353
   Specific Date=2014-01-01T10:10:30
   Current Date in IST=2018-05-29T16:40:00.353
   10000th second time from 01/01/1970= 1970-01-01T02:46:40
   ```

4. java.time.Instant：Instant类是用在机器可读的时间格式上的，它以Unix时间戳的形式存储日期时间，我们来看一个简单的程序

   ```java
   @Test
   public void testTimestampForInstant() {
       //Current timestamp
       Instant timestamp = Instant.now();
       System.out.println("Current Timestamp = " + timestamp);
   
       //Instant from timestamp
       Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli());
       System.out.println("Specific Time = " + specificTime);
   
       //Duration example
       Duration thirtyDay = Duration.ofDays(30);
       System.out.println(thirtyDay);
   }
   
   ```

5. 日期API工具：我们早些时候提到过，大多数日期/时间API类都实现了一系列工具方法，如：加/减天数、周数、月份数，等等。还有其他的工具方法能够使用TemporalAdjuster调整日期，并计算两个日期间的周期。

   ```java
   
   @Test
   public void testDateTool() {
       LocalDate today = LocalDate.now();
   
       //Get the Year, check if it's leap year
       System.out.println("Year " + today.getYear() + " is Leap Year? " + today.isLeapYear());
   
       //Compare two LocalDate for before and after
       System.out
           .println("Today is before 01/01/2015? " + today.isBefore(LocalDate.of(2015, 1, 1)));
   
       //Create LocalDateTime from LocalDate
       System.out.println("Current Time=" + today.atTime(LocalTime.now()));
   
       //plus and minus operations
       System.out.println("10 days after today will be " + today.plusDays(10));
       System.out.println("3 weeks after today will be " + today.plusWeeks(3));
       System.out.println("20 months after today will be " + today.plusMonths(20));
   
       System.out.println("10 days before today will be " + today.minusDays(10));
       System.out.println("3 weeks before today will be " + today.minusWeeks(3));
       System.out.println("20 months before today will be " + today.minusMonths(20));
   
       //Temporal adjusters for adjusting the dates
       System.out.println(
           "First date of this month= " + today.with(TemporalAdjusters.firstDayOfMonth()));
       LocalDate lastDayOfYear = today.with(TemporalAdjusters.lastDayOfYear());
       System.out.println("Last date of this year= " + lastDayOfYear);
   
       Period period = today.until(lastDayOfYear);
       System.out.println("Period Format= " + period);
       System.out.println("Months remaining in the year= " + period.getMonths());
   }
   ```

6. 解析和格式化：将一个日期格式转换为不同的格式，之后再解析一个字符串，得到日期时间对象，这些都是很常见的。我们来看一下简单的例子。

   ```java
   @Test
   public void testFormat() {
       //Format examples
       LocalDate date = LocalDate.now();
       //default format
       System.out.println("Default format of LocalDate=" + date);
       //specific format
       System.out.println(date.format(DateTimeFormatter.ofPattern("d::MMM::uuuu")));
       System.out.println(date.format(DateTimeFormatter.BASIC_ISO_DATE));
   
       LocalDateTime dateTime = LocalDateTime.now();
       //default format
       System.out.println("Default format of LocalDateTime=" + dateTime);
       //specific format
       System.out.println(dateTime.format(DateTimeFormatter.ofPattern("d::MMM::uuuu HH::mm::ss")));
       System.out.println(dateTime.format(DateTimeFormatter.BASIC_ISO_DATE));
   
       Instant timestamp = Instant.now();
       //default format
       System.out.println("Default format of Instant=" + timestamp);
   
       //Parse examples
       LocalDateTime dt = LocalDateTime.parse("27::五月::2014 21::39::48",
           DateTimeFormatter.ofPattern("d::MMM::uuuu HH::mm::ss"));
       System.out.println("Default format after parsing = " + dt);
   }
   ```

#### Jackson序列化LocalDate与Springboot集成

**问题**

LocalDate可以很友好的toString为`YYYY-MM-dd`的格式，很适合我当前的业务，但当我把它丢到json的时候，瞬间解体了：

```
{
  "year": 2018,
  "month": "AUGUST",
  "era": "CE",
  "dayOfMonth": 1,
  "dayOfWeek": "TUESDAY",
  "dayOfYear": 213,
  "leapYear": false,
  "monthValue": 8,
  "chronology": {
      "id":"ISO",
      "calendarType":"iso8601"
   }
}
```

可我想要的是yyyy-MM-dd啊。加上jackson format试一试，也不行。

```
@JsonFormat(shape = JsonFormat.Shape.STRING,
        pattern = "yyyy-MM-dd", timezone = "Asia/Shanghai")
```

难道要手动实现JsonSerializer? google之，果然有人解决了。

##### 解决

添加

```
<dependency>
  <groupId>com.fasterxml.jackson.datatype</groupId>
  <artifactId>jackson-datatype-jsr310</artifactId>
  <version>2.8.6</version>
</dependency>
```

用法

```
@Test
public void testJsonFormat() throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    LocalDate date = LocalDate.of(2018, 5, 5);
    String dateStr = mapper.writeValueAsString(date);
    Assert.assertEquals("\"2018-05-05\"", dateStr);

    LocalDateTime dateTime = LocalDateTime.of(2018, 5, 5, 1, 1, 1);
    Assert.assertEquals("\"2018-05-05T01:01:01\"", mapper.writeValueAsString(dateTime));
}
```

然而，在Springboot中，默认提供了ObjectMapper，我又不想自定义。

1. Springboot中使用

   同样把上述jar加入依赖。然后修改配置文件，新增

   ```
   spring:
     jackson:
       serialization:
         WRITE_DATES_AS_TIMESTAMPS: false
   ```

   这样可以直接使用LocalDate，不用单独JsonFormat就可以实现自己的功能了。

2. LocalDateTime序列化

   ```java
   @Configuration
   public class LocalDateTimeSerializerConfig {
       @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
       private String pattern;
   
       @Bean
       public LocalDateTimeSerializer localDateTimeDeserializer() {
           return new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
       }
   
       @Bean
       public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
           return builder -> builder.serializerByType(LocalDateTime.class, localDateTimeDeserializer());
       }
   
   }
   ```

   