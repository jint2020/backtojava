# Java 8 日期时间 API 详解

Java 8 引入了全新的日期时间 API（java.time 包），这个新的 API 解决了旧版日期时间 API 的诸多问题。本文将详细介绍新 API 的使用方法和最佳实践。

## 为什么需要新的日期时间 API？

旧版日期时间 API（java.util.Date 和 java.util.Calendar）存在以下问题：
1. 非线程安全
2. 设计不合理（如年份从1900开始，月份从0开始）
3. 时区处理麻烦
4. API 不直观
5. 格式化和解析操作复杂

## 核心类介绍

### 1. LocalDate - 处理日期
```java
// 创建日期
LocalDate today = LocalDate.now(); // 当前日期
LocalDate date = LocalDate.of(2025, 4, 24); // 指定日期
LocalDate parsed = LocalDate.parse("2025-04-24"); // 解析字符串

// 获取信息
int year = date.getYear(); // 获取年
Month month = date.getMonth(); // 获取月
int day = date.getDayOfMonth(); // 获取日
DayOfWeek dayOfWeek = date.getDayOfWeek(); // 获取星期
int dayOfYear = date.getDayOfYear(); // 获取年中的第几天

// 日期操作
LocalDate tomorrow = date.plusDays(1); // 增加天数
LocalDate lastMonth = date.minusMonths(1); // 减少月数
LocalDate nextYear = date.plus(1, ChronoUnit.YEARS); // 增加年数
```

### 2. LocalTime - 处理时间
```java
// 创建时间
LocalTime now = LocalTime.now(); // 当前时间
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
LocalTime parsed = LocalTime.parse("13:45:20"); // 解析字符串

// 获取信息
int hour = time.getHour(); // 获取小时
int minute = time.getMinute(); // 获取分钟
int second = time.getSecond(); // 获取秒

// 时间操作
LocalTime plusHours = time.plusHours(2); // 增加小时
LocalTime minusMinutes = time.minusMinutes(30); // 减少分钟
```

### 3. LocalDateTime - 处理日期和时间
```java
// 创建日期时间
LocalDateTime now = LocalDateTime.now(); // 当前日期时间
LocalDateTime dateTime = LocalDateTime.of(2025, 4, 24, 13, 45, 20); // 指定日期时间
LocalDateTime fromDate = LocalDateTime.of(date, time); // 从LocalDate和LocalTime创建

// 转换
LocalDate date = dateTime.toLocalDate(); // 转换为LocalDate
LocalTime time = dateTime.toLocalTime(); // 转换为LocalTime

// 操作
LocalDateTime future = dateTime.plusDays(1).minusHours(2); // 链式操作
```

### 4. ZonedDateTime - 处理时区
```java
// 创建带时区的日期时间
ZonedDateTime nowInSystemTZ = ZonedDateTime.now(); // 系统默认时区
ZonedDateTime nowInNY = ZonedDateTime.now(ZoneId.of("America/New_York")); // 指定时区
ZonedDateTime nowInShanghai = ZonedDateTime.now(ZoneId.of("Asia/Shanghai")); // 上海时区

// 时区转换
ZonedDateTime converted = nowInNY.withZoneSameInstant(ZoneId.of("Asia/Shanghai"));

// 获取所有可用时区
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
```

### 5. Instant - 处理时间戳
```java
// 创建时间戳
Instant now = Instant.now(); // 当前时间戳
Instant specific = Instant.ofEpochMilli(1234567890000L); // 从毫秒创建

// 操作
Instant future = now.plus(Duration.ofHours(5)); // 增加5小时
Instant past = now.minus(Duration.ofDays(7)); // 减少7天

// 比较
boolean isAfter = future.isAfter(now); // 比较时间戳
```

### 6. Duration 和 Period - 处理时间段
```java
// Duration - 处理基于时间的间隔
Duration fiveHours = Duration.ofHours(5);
Duration tenMinutes = Duration.ofMinutes(10);
Duration fromDays = Duration.ofDays(1);

// Period - 处理基于日期的间隔
Period tenDays = Period.ofDays(10);
Period twoMonths = Period.ofMonths(2);
Period customPeriod = Period.of(1, 2, 3); // 1年2月3天
```

## 日期时间格式化

### 1. DateTimeFormatter
```java
// 预定义格式器
DateTimeFormatter isoFormatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
String formatted = LocalDateTime.now().format(isoFormatter);

// 自定义格式器
DateTimeFormatter customFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String customFormatted = LocalDateTime.now().format(customFormatter);

// 解析
LocalDateTime parsed = LocalDateTime.parse("2025-04-24 13:45:20", customFormatter);

// 本地化格式器
DateTimeFormatter localizedFormatter = DateTimeFormatter
    .ofLocalizedDateTime(FormatStyle.MEDIUM)
    .withLocale(Locale.CHINA);
```

## 日期时间计算

### 1. 日期比较
```java
LocalDate date1 = LocalDate.of(2025, 4, 24);
LocalDate date2 = LocalDate.of(2025, 4, 25);

boolean isBefore = date1.isBefore(date2);
boolean isAfter = date1.isAfter(date2);
boolean isEqual = date1.isEqual(date2);
```

### 2. 时间间隔计算
```java
// 计算两个日期之间的天数
long daysBetween = ChronoUnit.DAYS.between(date1, date2);

// 计算两个时间之间的小时数
long hoursBetween = ChronoUnit.HOURS.between(time1, time2);

// 计算年龄
LocalDate birthDate = LocalDate.of(1990, 1, 1);
long age = ChronoUnit.YEARS.between(birthDate, LocalDate.now());
```

### 3. 日期调整器
```java
// 获取下一个周日
LocalDate nextSunday = date.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));

// 获取本月最后一天
LocalDate lastDayOfMonth = date.with(TemporalAdjusters.lastDayOfMonth());

// 自定义调整器
TemporalAdjuster firstDayOfNextMonth = TemporalAdjusters.firstDayOfNextMonth();
LocalDate firstOfNextMonth = date.with(firstDayOfNextMonth);
```

## 最佳实践

### 1. 选择合适的类
```java
// 仅需要日期
LocalDate date = LocalDate.now();

// 仅需要时间
LocalTime time = LocalTime.now();

// 需要日期和时间
LocalDateTime dateTime = LocalDateTime.now();

// 需要处理时区
ZonedDateTime zonedDateTime = ZonedDateTime.now();
```

### 2. 使用不可变对象特性
```java
// 正确的方式
LocalDate date = LocalDate.now();
LocalDate newDate = date.plusDays(1); // 创建新对象

// 错误的方式
date.plusDays(1); // 这行代码不会改变date的值
```

### 3. 格式化最佳实践
```java
// 缓存格式器
private static final DateTimeFormatter FORMATTER = 
    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

// 使用缓存的格式器
String formatted = LocalDateTime.now().format(FORMATTER);
```

## 性能考虑

1. **缓存 DateTimeFormatter**：格式器的创建成本较高
2. **使用合适的时间精度**：根据需求选择合适的时间精度
3. **注意时区转换的性能开销**

## 常见问题解决

### 1. 与旧 API 转换
```java
// Date 转 Instant
Instant instant = new Date().toInstant();

// Instant 转 Date
Date date = Date.from(instant);

// Calendar 转 Instant
Instant instant = calendar.toInstant();
```

### 2. 处理遗留系统
```java
// 转换为 java.sql.Date
java.sql.Date sqlDate = java.sql.Date.valueOf(localDate);

// 转换为 java.sql.Timestamp
Timestamp timestamp = Timestamp.valueOf(localDateTime);
```

## 总结

Java 8 的日期时间 API 提供了：
1. 更清晰的 API 设计
2. 不可变对象，保证线程安全
3. 更好的时区支持
4. 标准化的日期时间操作

建议在新项目中优先使用新的日期时间 API，它能帮助我们写出更加清晰、安全的代码。