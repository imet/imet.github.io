---
title: "java日志框架"
date: 2018-07-17T16:47:00+08:00
lastmod: 2018-07-17T16:47:00+08:00
draft: false
keywords:
- log4j
- log4j2
- slf4j
description: "java 日志框架"
tags:
- log4j
- slf4j
- log4j2
categories:
- java框架
author: "imet"
---

日志组件这么多，log4j, log4j2, slf4j 等傻傻分不清楚，如何选择相关的日志组件 ？

<!--more-->

## 简单的总结

- 现在主流的都推荐使用 slf4j 作为日志门面
- 在一个应用程序里面，slf4j 有且只能有一个 implementation
- log4j2 性能更高: 有更优秀的锁机制，LMAX Disruptor 队列，"无垃圾"机制
- 有了 slf4j, log4j2 后，需要把其他的日志组件给排除掉，然后将其他的日志重定向到 slf4j, 或者 log4j2

### slf4j binding 一张图

![slf4j binding]( http://p0.img.imet.top/op/20180718141814_slf4j_impl.png)

### slf4j 使用 log4j1.2 作为实现，然后将其他日志转到 slf4j

![slf4j_with_redirection]( http://p0.img.imet.top/op/20180718141933_slf4j_with_redirection.png)


### 给出用 log4j2 一个日志相关的 maven 配置

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

## slf4j提供的适配库和桥接库

### 适配库：

- slf4j-log4j12：使用log4j-1.2作为日志输出服务
- slf4j-jdk14：使用java.util.logging作为日志输出服务
- slf4j-jcl：使用JCL作为日志输出服务
- slf4j-simple：日志输出至System.err
- slf4j-nop：不输出日志
- log4j-slf4j-impl：使用log4j2作为日志输出服务
- logback: logback天然与slf4j适配，不需要额外引入适配库（毕竟是一个作者写的）

### 桥接库：

- log4j-over-slf4j：将使用log4j api输出的日志桥接至slf4j
- jcl-over-slf4j：将使用JCL api输出的日志桥接至slf4j
- jul-to-slf4j：将使用java.util.logging输出的日志桥接至slf4j
- log4j-to-slf4j：将使用log4j2输出的日志桥接至slf4j

> 题外话：slf4j唯独没有提供log4j2的适配库和桥接库，log4j-slf4j-impl和log4j-to-slf4j都是Apache Logging自己开发的，看样子Ceki和Apache Logging的梁子真的很深啊……倒是Apache没有端架子，可能也是因为slf4j太火了吧

## log4j2 提供的适配库

### 桥接库及绑定库

- **log4j-jcl**: 将使用 jcl api 输出的日志桥接到 log4j2
  * summary: The Commons Logging Bridge allows applications coded to the Commons Logging API to use Log4j 2 as the implementation.

- **log4j-1.2-api**: 将 log4j 1.2 api 输出的日志桥接到 log4j2
  * desc: The Log4j 1.2 Bridge allows applications coded to use Log4j 1.2 API to use Log4j 2 instead

- **log4j-jul**: 将 jdk jul 日志使用 log4j2 来输出
  * summary: The JDK Logging Adapter is a custom implementation of java.util.logging.LogManager that uses Log4j. This adapter can be used with either the Log4j API or Log4j Core

- **log4j-slf4j-impl**：使用 log4j2 作为日志输出服务
  * summary: The Log4j 2 SLF4J Binding allows applications coded to the SLF4J API to use Log4j 2 as the implementation.

- **log4j-to-slf4j**: 将 log4j2 日志输出的日志桥接到 slf4j

> log4j-slf4j-impl 不要和 log4j-to-slf4j 同时使用，不然后果很严重

## 参考
1. csdn blog: [java日志组件介绍（common-logging，log4j，slf4j，logback ）][1]
2. 简书: [该让log4j退休了 - 论Java日志组件的选择][2]
3. 网易lede: [日志工具现状调研][3]
4. 网易lede: [Log4j 2.x 日志升级的详细流程（针对各种混合使用Log4j 1.x，Logback，JCL,JUL等）][4]
5. log4j2 runtime-dependencies [log4j2 相关 jar 及依赖关系][5]

[1]: https://blog.csdn.net/yycdaizi/article/details/8276265 "java日志组件介绍（common-logging，log4j，slf4j，logback ）"
[2]: https://www.jianshu.com/p/85d141365d39 "该让log4j退休了 - 论Java日志组件的选择 - on 简书"
[3]: http://tech.lede.com/2017/02/06/rd/server/log4jSearch/ "日志工具现状调研"
[4]: http://tech.lede.com/2017/08/28/rd/server/Log4j2Update/ "Log4j 2.x 日志升级的详细流程（针对各种混合使用Log4j 1.x，Logback，JCL,JUL等）"
[5]: http://logging.apache.org/log4j/2.x/runtime-dependencies.html "log4j2 相关 jar 及依赖关系"
