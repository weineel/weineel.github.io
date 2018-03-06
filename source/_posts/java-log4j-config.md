---
title: log4j配置示例, 资料汇总
date: 2017.11.09 15:28:05
tags: 
    - java
    - log4j
categories: java
---

----
日志是一个很庞大的话题，有机会要仔细研究下。

<!-- more -->

## 配置示例log4j1.2.17
```
# log4j.properties
# debug, D, C, E：第一个是日志输出级别，后面都是输出目的地
log4j.rootLogger = debug, D, C, E

### console ###
log4j.appender.C = org.apache.log4j.ConsoleAppender
log4j.appender.C.Target = System.out
log4j.appender.C.layout = org.apache.log4j.PatternLayout
log4j.appender.C.layout.ConversionPattern = [financeTown-web][%p] [%-d{yyyy-MM-dd HH:mm:ss}] %C.%M(%L) | %m%n
log4j.appender.C.file.encoding=UTF-8

### log file ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.Append = true
# 对使用这个appender的日志二次过滤。
log4j.appender.D.Threshold = DEBUG
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = [financeTown-web][%p] [%-d{yyyy-MM-dd HH:mm:ss}] %C.%M(%L) | %m%n
log4j.appender.D.file.encoding=UTF-8
log4j.appender.D.File = ../logs/financeTown-web
# 按日期每天一个日志，当天是日志文件名为没有日期的文件名，会在后一天另存为追加为有以下日期格式的文件。
log4j.appender.D.DatePattern='-'yyyy-MM-dd.'log'
# 每分钟生成一个log文件
# log4j.appender.D.DatePattern='-'yyyy-MM-dd-HH-mm.'log'

### exception ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = [financeTown-web][%p] [%-d{yyyy-MM-dd HH:mm:ss}] %C.%M(%L) | %m%n
log4j.appender.E.file.encoding=UTF-8
log4j.appender.E.File = ../logs/financeTown-web_error
log4j.appender.E.DatePattern='-'yyyy-MM-dd.'log'

```
## 参考
项目原因，研究了下log4j，之前只是用已有的配置，没有自己配置过。以下是我参考的一些文章。
* [java常用日志库](https://www.cnblogs.com/chenhongliang/p/5312517.html)，概括了java日志发展的历史，值得一看，但是主要讲了Slf4j，算是篇软文。
* [apache 官方](http://logging.apache.org/log4j/1.2/)
* [Log4j1.2配置详解](http://www.cnblogs.com/zerotomax/p/7400085.html)
* [配置日志文件相对路径](http://lzhw1985.iteye.com/blog/1940880)
* [log4j框架logger的继承关系以及使用场景](http://blog.csdn.net/aitangyong/article/details/50392227)
