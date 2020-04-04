---
layout:     post
title:      "simpledateformat"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-21 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - Java并发编程
---

* content
{:toc}

## SimpleDateFormat

java.text.SimpleDateFormat，是线程非安全,原因在于SimpleDateFormat中，使用到了Calendar类对象，在执行parse() 和format()方法时，调用calendar.setTime(date)方法。那么问题来了：多线程环境下,线程非安全。 
解决方案：
1）每次使用，都创建新实例； 
2）使用三方类库。

## 创建新实例
```
	public static String formatDate(Date date){
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        return simpleDateFormat.format(date);
    }

	public static Date parseDate(String dateStr) throws ParseException {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        return simpleDateFormat.parse(dateStr);
    }
```
仅在需要用到的地方创建一个新的实例，就没有线程安全问题，不过也加重了创建对象的负担，会频繁地创建和销毁对象，效率较低。

## Synchronized方式
```
 private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

    public static String formatDate(Date date) {
        synchronized (sdf) {
            return sdf.format(date);
        }
    }
    
    public static Date parseDate(String dateStr) throws ParseException {
        synchronized (sdf) {
            return sdf.parse(dateStr);
        }
    }
```
synchronized往上一套也可以解决线程安全问题，缺点自然就是并发量大的时候会对性能有影响，线程阻塞。

## ThreadLocal
```
private static final ThreadLocal<DateFormat> DATE_FORMAT = new ThreadLocal<DateFormat>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd");
        }
    };

    private static final ThreadLocal<DateFormat> DATETIME_FAT = new ThreadLocal<DateFormat>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static String formatDate(Date date) {
        return DATE_FORMAT.get().format(date);
    }
    
    public static Date parseDate(String dateStr) throws ParseException {
        return DATE_FORMAT.get().parse(dateStr);
    }
```
ThreadLocal可以确保每个线程都可以得到单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。

## DateTimeFormatter
基于JDK1.8的DateTimeFormatter,也是《阿里巴巴开发手册》给我们的解决方案
```
private static final DateTimeFormatter DATE_TIME_FORMATTER = 
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static String formatLocalDateTime(LocalDateTime localDateTime) {
        return DATE_TIME_FORMATTER.format(localDateTime);
    }

    public static LocalDateTime parseLocalDateTime(String dateStr) {
        return LocalDateTime.parse(dateStr, DATE_TIME_FORMATTER);
    }
```
DateTimeFormatter源码上作者也加注释说明了，他的类是不可变的，并且是线程安全的。
