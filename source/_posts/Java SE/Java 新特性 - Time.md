---
title: Java 新特性之 Time
date: 2019-05-10 10:49:28
tags: Java8
categories:
 -Java
---

## 日期类 LocalDate 

能够简单的获取当天的日期，并且可以方便的对日期进行加减。

他是通过静态方法或者 from/of 等方法创建对象的。

这个类不存储时区，所以他没有时区的概念，如果需要时区的话需要使用 `ZonedDateTime`

这个类只能进行日期的相关操作，没有具体的时间。

下面介绍常用的几个方法

### atTime

生成一个带有时间的日期，返回结果是 `LocalDateTime` 所以很明显这个就是带有时间的日期类。

```java
LocalDateTime time = LocalDate.now().atTime(12, 3, 22,232);
System.out.println(time); //2019-05-10T12:03:22.000000232
```

### compareTo()

比较两个时间返回的值是 int ，大于 0 则是比较者大，否则是被比较者大，为0相等。

### format()

格式化时间，需要传递一个时间的 formatter 类型为 `[DateTimeFormatter]` 

### parse()

同上第一个参数是解析的字符串，第二个就是传入formatter

### plus/minus

对日期进行加减，第一个参数传入的是 int 第二个就是单位，单位使用 `ChronoUnit` 枚举类型

```java
public class LocalDate1 {

    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        // 添加时间
        LocalDate i22day = today.plusDays(22);
        LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
        // 减去时间
        LocalDate s22day = today.minus(22, ChronoUnit.DAYS);
        LocalDate yesterday = tomorrow.minusDays(2);

        System.out.println(today);
        System.out.println(tomorrow);
        System.out.println(yesterday);
        System.out.println(i22day);
        System.out.println(s22day);

        LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
        DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
        System.out.println(dayOfWeek);    // FRIDAY

        DateTimeFormatter germanFormatter =
                DateTimeFormatter
                        .ofLocalizedDate(FormatStyle.MEDIUM)
                        .withLocale(Locale.GERMAN);

        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy/mm/dd hh:mm:ss");
        LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
        // System.out.println(LocalDate.parse("2014/02/01 12:03:22", dateTimeFormatter));
        System.out.println(xmas);   // 2014-12-24


        System.out.println(LocalDate.now().atStartOfDay());

        LocalDateTime time = LocalDate.now().atTime(12, 3, 22,232);
        System.out.println(time);
    }
```



## 时间 LocalDateTime

这个和上面类似是一个时间类，不仅仅有日期还是时间。

### atZone

产生带有时区日期

### format

格式化时间

### of

从一个时间创建对象 `of(int year, int month, int dayOfMonth, int hour, int minute, int second)`

### parse

解析时间 `**parse**([CharSequence]text, [DateTimeFormatter]`

```java
public class LocalDateTime1 {

    public static void main(String[] args) {

        LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

        DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
        System.out.println(dayOfWeek);      // WEDNESDAY

        Month month = sylvester.getMonth();
        System.out.println(month);          // DECEMBER

        long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
        System.out.println(minuteOfDay);    // 1439

        Instant instant = sylvester
                .atZone(ZoneId.systemDefault())
                .toInstant();

        Date legacyDate = Date.from(instant);
        System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014


        DateTimeFormatter formatter =
                DateTimeFormatter
                        .ofPattern("MMM dd, yyyy - HH:mm");

        LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
        String string = parsed.format(formatter);
        System.out.println(string);     // Nov 03, 2014 - 07:13
    }

}
```



## LocalTime

这个类是上面的LocaDateTime中的 Time 部分，也就是只具有时间没有日期。

```
public class LocalDate1 {

    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        // 添加时间
        LocalDate i22day = today.plusDays(22);
        LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
        // 减去时间
        LocalDate s22day = today.minus(22, ChronoUnit.DAYS);
        LocalDate yesterday = tomorrow.minusDays(2);

        System.out.println(today);
        System.out.println(tomorrow);
        System.out.println(yesterday);
        System.out.println(i22day);
        System.out.println(s22day);

        LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
        DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
        System.out.println(dayOfWeek);    // FRIDAY

        DateTimeFormatter germanFormatter =
                DateTimeFormatter
                        .ofLocalizedDate(FormatStyle.MEDIUM)
                        .withLocale(Locale.GERMAN);

        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy/mm/dd hh:mm:ss");
        LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
        // System.out.println(LocalDate.parse("2014/02/01 12:03:22", dateTimeFormatter));
        System.out.println(xmas);   // 2014-12-24


        System.out.println(LocalDate.now().atStartOfDay());

        LocalDateTime time = LocalDate.now().atTime(12, 3, 22,232);
        System.out.println(time);
    }
```



## Clock

这个类可以获取当前的时间戳

```java
public class Clock1 {
    public static void main(String[] args) {
        Clock millis = Clock.tickMillis(ZoneOffset.UTC);
        Clock seconds = Clock.tickSeconds(ZoneOffset.UTC);
        Clock minutes = Clock.tickMinutes(ZoneOffset.UTC);
        // 精确到毫秒的时间戳
        System.out.println(millis.millis());
        // 精确到秒的时间戳
        System.out.println(seconds.millis());
        // 精确到分钟的时间戳
        System.out.println(minutes.millis());
        // 精确到毫秒的时间
        System.out.println(millis.instant());
        // 精确到秒的时间
        System.out.println(seconds.instant());
        // 精确到分钟的时间
        System.out.println(minutes.instant());
    }
}
```

上面的输出结果是：

```
1557567339397
1557567339000
1557567300000
2019-05-11T09:35:39.397Z
2019-05-11T09:35:39Z
2019-05-11T09:35:00Z
```

## Map初始化小技巧

再看Clock类中的 ZoneId 接口的时候发现了这个接口中使用了一个Map，然对Map做了初始化。一般初始化必须使用 put 放数据，这里采用了 ofEntries 来初始化多组数据。

```java
public static final Map<String, String> SHORT_IDS = Map.ofEntries(
        entry("ACT", "Australia/Darwin"),
        entry("AET", "Australia/Sydney"),
        entry("AGT", "America/Argentina/Buenos_Aires"),
        entry("ART", "Africa/Cairo"),
        entry("AST", "America/Anchorage"),
        entry("BET", "America/Sao_Paulo"),
        entry("BST", "Asia/Dhaka"),
        entry("CAT", "Africa/Harare"),
        entry("CNT", "America/St_Johns"),
        entry("CST", "America/Chicago"),
        entry("CTT", "Asia/Shanghai"),
        entry("EAT", "Africa/Addis_Ababa"),
        entry("ECT", "Europe/Paris"),
        entry("IET", "America/Indiana/Indianapolis"),
        entry("IST", "Asia/Kolkata"),
        entry("JST", "Asia/Tokyo"),
        entry("MIT", "Pacific/Apia"),
        entry("NET", "Asia/Yerevan"),
        entry("NST", "Pacific/Auckland"),
        entry("PLT", "Asia/Karachi"),
        entry("PNT", "America/Phoenix"),
        entry("PRT", "America/Puerto_Rico"),
        entry("PST", "America/Los_Angeles"),
        entry("SST", "Pacific/Guadalcanal"),
        entry("VST", "Asia/Ho_Chi_Minh"),
        entry("EST", "-05:00"),
        entry("MST", "-07:00"),
        entry("HST", "-10:00")
    );
```

