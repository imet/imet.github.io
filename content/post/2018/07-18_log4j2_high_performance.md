---
title: "log4j2 高性能"
date: 2018-07-18T11:48:09+08:00
lastmod: 2018-07-18T11:48:09+08:00
draft: false
keywords:
- log4j
- log4j2
- slf4j
description: "log4j2 日志框架调优及高性能"
tags:
- log4j
- slf4j
- log4j2
categories:
- java框架
author: "imet"
---

## 开篇词

log4j2 日志性能在某些场景下要优于 logback 等其他日志组件，带来了非常大的性能提升，究竟有多大呢？

这几个新的特性如下:

- [Asynchronous Loggers][6] based on the [LMAX Disruptor library]
- [Garbage-free][8]
- Plugin system, 方便添加 appenders, filters, layouts, lookups 等

<!--more-->

## 高性能的几个表现对比 (High performance)

### 1. 日志组件之间的对比

**结论：log4j2 all async 胜出，异步是同步的6-10倍**

主要得益于:

- [lock-free data structure][9]

![Logging Library Performance Comparison]( http://pb56ttwyu.bkt.clouddn.com/blog/img/20180718145851_log_lib_compare.png)

(refer: [log4j2 performance][7])

### 2. 异步日志场景下是否记录调用栈

**结论：不记录相关的 class, line num 这些信息的话，性能有 30~100 的提升**

如果日志 pattern 中打印以下信息的话:  %C or $class, %F or %file, %l or %location, %L or %line, %M or %method
将会极大的影响日志的性能，日志组件需要打印当前栈的快照，遍历 stack trace 然后找到 location 等相关信息

![Asynchronous Logging with Caller Location Information]( http://pb56ttwyu.bkt.clouddn.com/blog/img/20180718151032_performance_impact_of_catpuring_caller_location.png)

(refer: [log4j2 performance][7])

### 3. 同步场景下 garbage free

大部分情况下 garbage free 的性能要优

![sync log compare]( http://pb56ttwyu.bkt.clouddn.com/blog/img/20180718173203_sync_logging_compare.png)

(refer: [log4j2 performance][7], [garbagefree][8])

## log4j2 相关的配置

### maven pom 相关的配置

.pom.xml
```xml
  <dependencies>
    <!-- 日志门面 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>

    <!-- log4j2 作为 slf4j 的实现 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.10.0</version>
    </dependency>

    <!-- log4j2 核心 jar -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.10.0</version>
    </dependency>

    <!-- log4j2 核心 jar -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.10.0</version>
    </dependency>

    <!-- 将老版本的 log4j 1.2 升级到 log4j2  -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-1.2-api</artifactId>
        <version>2.10.0</version>
    </dependency>

    <!--&lt;!&ndash; log4j2 jul 的实现  &ndash;&gt;-->
    <!--<dependency>-->
        <!--<groupId>org.apache.logging.log4j</groupId>-->
        <!--<artifactId>log4j-jul</artifactId>-->
        <!--<version>2.10.0</version>-->
    <!--</dependency>-->

    <!-- 使用log4j2的AsyncLogger需要包含disruptor -->
    <dependency>
        <groupId>com.lmax</groupId>
        <artifactId>disruptor</artifactId>
        <version>3.3.7</version>
    </dependency>

    <!-- 需要再把其他依赖里面的 slf4j implement 给排除掉，已经依赖的实现日志包 -->
    <dependency>
      <groupId>xxx</groupId>
      <artifactId>xxx-yy</artifactId>
      <version>zz</version>
      <exclusions>
        <exclusion>
            <artifactId>slf4j-log4j12</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
        </exclusion>
      </exclusions>
    </dependency>

  </dependencies>
```

### log4j2 日志相关的配置

.log4j2.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="LOG_HOME">./logs</Property>
    </Properties>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%level [%date{HH:mm:ss.SSS}][%thread][%class][%line]:%message%n"/>
        </Console>

        <RollingRandomAccessFile name="infoLog" fileName="${LOG_HOME}/info.log"
                                 filePattern="${LOG_HOME}/info.%d{yyyy-MM-dd}.log.gz" immediateFlush="false" append="true">
            <PatternLayout pattern="[%X][%date{yyyy-MM-dd HH:mm:ss.SSS}][%thread][%level]:%message%n"/>
            <Filters>
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="warn" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
                <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingRandomAccessFile>
        <RollingRandomAccessFile name="warnLog" fileName="${LOG_HOME}/warn.log"
                                 filePattern="${LOG_HOME}/warn.%d{yyyy-MM-dd}.log.gz" immediateFlush="false" append="true">
            <Filters>
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <PatternLayout pattern="[%X][%date{yyyy-MM-dd HH:mm:ss.SSS}][%thread][%level][%class][%line]:%message%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingRandomAccessFile>
        <RollingRandomAccessFile name="errorLog" fileName="${LOG_HOME}/error.log"
                                 filePattern="${LOG_HOME}/error.%d{yyyy-MM-dd}.log.gz" immediateFlush="false" append="true">
            <Filters>
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <PatternLayout pattern="[%X][%date{yyyy-MM-dd HH:mm:ss.SSS}][%thread][%level][%class][%line]:%message%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingRandomAccessFile>

    </Appenders>

    <Loggers>

        <!-- 指定相关的 logger name 的日志级别 -->
        <logger name="org.springframework" level="info" additivity="false">
            <appender-ref ref="infoLog"/>
            <appender-ref ref="warnLog"/>
            <appender-ref ref="errorLog"/>
        </logger>

        <!-- 剩余其他的日志级别 -->
        <Root level="warn" includeLocation="true">
            <appender-ref ref="infoLog"/>
            <appender-ref ref="warnLog"/>
            <appender-ref ref="errorLog"/>
        </Root>

    </Loggers>

</Configuration>

```

.log4j2.component.properties
```bash
# 设置全异步的打日志方式， 就不需要在命令行加 -Dlog4j2.contextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector 参数了
log4j2.contextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
# size = 1024 * 1024
log4j2.asyncLoggerRingBufferSize=1048576
# discard not to block the queue
log4j2.asyncQueueFullPolicy=Discard
# 默认是在 INFO 级别去 discard
log4j2.discardThreshold=INFO
```

## 参考
1. csdn blog: [java日志组件介绍（common-logging，log4j，slf4j，logback ）][1]
2. 简书: [该让log4j退休了 - 论Java日志组件的选择][2]
3. 网易lede: [日志工具现状调研][3]
4. 网易lede: [Log4j 2.x 日志升级的详细流程（针对各种混合使用Log4j 1.x，Logback，JCL,JUL等）][4]
5. log4j2 runtime-dependencies: [log4j2 相关 jar 及依赖关系][5]
6. log4j2 async: [Asynchronous Loggers][6]
7. log4j2 performance: [log4j2 performance][7]
8. log4j2 garbagefree: [garbagefree][8]
9. lock-free by disruptor: [lock-free data structure][9]
10. log4j 配置文件: [log4j2 configuration][10]
11. [log4j2异步注意事项][11]
12. [log4j2性能剖析][12]
13. [log4j2异步Logger][13]



[1]: https://blog.csdn.net/yycdaizi/article/details/8276265 "java日志组件介绍（common-logging，log4j，slf4j，logback ）"
[2]: https://www.jianshu.com/p/85d141365d39 "该让log4j退休了 - 论Java日志组件的选择 - on 简书"
[3]: http://tech.lede.com/2017/02/06/rd/server/log4jSearch/ "日志工具现状调研"
[4]: http://tech.lede.com/2017/08/28/rd/server/Log4j2Update/ "Log4j 2.x 日志升级的详细流程（针对各种混合使用Log4j 1.x，Logback，JCL,JUL等）"
[5]: http://logging.apache.org/log4j/2.x/runtime-dependencies.html "log4j2 相关 jar 及依赖关系"
[6]: http://logging.apache.org/log4j/2.x/manual/async.html "Asynchronous Loggers for Low-Latency Logging"
[7]: https://logging.apache.org/log4j/2.x/performance.html "log4j2 performance"
[8]: http://logging.apache.org/log4j/2.x/manual/garbagefree.html "garbagefree"
[9]: https://lmax-exchange.github.io/disruptor/ "lock-free data structure by disruptor"
[10]: https://logging.apache.org/log4j/2.x/manual/configuration.htm "log4j2 configuration"
[11]: https://www.jianshu.com/p/82469047acbf "log4j2异步注意事项"
[12]: https://my.oschina.net/13426421702/blog/904249 "log4j2性能剖析"
[13]: https://blog.csdn.net/heyutao007/article/details/72773077 "log4j2异步Logger"
