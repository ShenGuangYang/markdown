# java date 8

## LocalDate、LocalTime







## 日期操作

传递`TemporalAdjuster`对象进行灵活的处理日期。

```java
LocalDate date1 = LocalDate.of(2014, 3, 18); 
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); 
LocalDate date3 = date2.with(lastDayOfMonth());
```

### **TemporalAdjuster**类中的工厂方法

| 方法名              | 描述                                   |
| :------------------ | :------------------------------------- |
| dayOfWeekInMonth    | 同一个月中每一周的第几天               |
| firstDayOfMonth     | 当月的第一天                           |
| firstDayOfNextMonth | 下月的第一天                           |
| firstDayOfNextYear  | 明年的第一天                           |
| firstDayOfYear      | 当年的第一天                           |
| firstInMonth        | 同一个月中，第一个符合星期几要求的值   |
| lastDayOfMonth      | 当月的最后一天                         |
| lastDayOfNextMonth  | 下月的最后一天                         |
| lastDayOfNextYear   | 明年的最后一天                         |
| lastDayOfYear       | 今年的最后一天                         |
| lastInMonth         | 同一个月中，最后一个符合星期几要求的值 |
|                     |                                        |
|                     |                                        |
|                     |                                        |



## 日期转换



```java
LocalDateTime localDateTime = LocalDateTime.now();
// LocalDateTime 转 String
String format = localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
// String 转 LocalDateTime
LocalDateTime parse = LocalDateTime.parse(format, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

```







