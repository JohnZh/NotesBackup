UTC 时间格式转换

---



# 时间格式

```
2021-01-06T14:00:00+09:00
2021-01-06T14:00:00.000+09:00
```

## 概念和字符串表达式

GMT 格林尼治平时（Greenwich MeanTime）是一个时区

UTC 世界标准时间 Coordinated Universal Time 是一个时间标准，ISO8601 是 UTC 的一种表示方式，其表达方式：

- yyyy-MM-dd'T'HH:mm:ss.SSSXXX：2018-05-23T16:05:52.123+08:00
- yyyy-MM-dd'T'HH:mm:ssXXX：2018-05-23T16:05:52+08:00
- yyyy-MM-dd HH:mm:ss：2018-05-24 00:05:52
- EEE MMM dd HH:mm:ss zzz yyyy：Thu May 24 00:05:52 CST 2018

> - T：分隔日期和时间
> - .sss：毫秒
> - Z：时区，可以是：Z（UFC）、+HH:mm、-HH:mm

## 例子和解释

```java
/**
     * 函数功能描述:UTC时间转本地时间格式
     * @param utcTime UTC时间
     * @param utcTimePatten UTC时间格式
     * @param localTimePatten   本地时间格式
     * @return 本地时间格式的时间
     * eg:utc2Local("2017-06-14T09:37:50.788+08:00", "yyyy-MM-dd'T'HH:mm:ss.SSSXXX", "yyyy-MM-dd HH:mm:ss.SSS")
     */
public static String utc2Local(String utcTime, String utcTimePatten, String localTimePatten) {
  SimpleDateFormat utcFormater = new SimpleDateFormat(utcTimePatten);
  utcFormater.setTimeZone(TimeZone.getTimeZone("UTC"));//时区定义并进行时间获取
  Date date = null;
  try {
    date = utcFormater.parse(utcTime);
  } catch (ParseException e) {
    e.printStackTrace();
    return utcTime;
  }
  SimpleDateFormat localFormater = new SimpleDateFormat(localTimePatten);
  localFormater.setTimeZone(TimeZone.getDefault());
  String localTime = localFormater.format(date.getTime());
  return localTime;
}
```

本地时间格式入参都是：yyyy-MM-dd HH:mm:ss

- 2018-05-23 16:05:52, yyyy-MM-dd HH:mm:ss：**2018-05-24 00:05:52**
  - 字符串中的‘T’可以用空格代替
  - 2018-05-23 16:05:52 就是 GMT 时间，2018-05-24 00:05:52 就是本地时间（东 8 区，时间要 + 8）
- 2018-05-23T16:05:52.000+00:00, yyyy-MM-dd'T'HH:mm:ss.SSSXXX：**2018-05-24 00:05:52**
  - 2018-05-23T16:05:52.000+00:00 GMT 时间，2018-05-24 00:05:52 本地时间
- 2018-05-23T16:05:52.000+08:00, yyyy-MM-dd'T'HH:mm:ss.SSSXXX：**2018-05-23 16:05:52**
  - 2018-05-23T16:05:52.000+08:00 已经指明了时区为东 8 区时间，因此结果是一样的
- 2018-05-23T16:05:52+08:00, yyyy-MM-dd'T'HH:mm:ssXXX：**2018-05-23 16:05:52**
  - 同理
- 2018-05-23T16:05:52+08:30, yyyy-MM-dd'T'HH:mm:ssX：**2018-05-23 16:05:52**
  - 由于只有一个 X，+08:30 只读取了 08
- 2018-05-23T16:05:52.123Z, yyyy-MM-dd'T'HH:mm:ss.SSS'Z'：**2018-05-24 00:05:52**



## 使用 ZoneDateTime 处理





