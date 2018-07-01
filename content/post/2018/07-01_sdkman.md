---
title: "sdkman"
date: 2018-07-01T18:44:43+08:00
lastmod: 2018-07-01T18:44:43+08:00
draft: true
keywords:
- SDKMAN
- groovy
- scala
- kotlin
-
description: "java 软件管理工具"
tags:
-
categories:
-
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams:
  enable: false
  options: ""

---

java 相关的一些软件如 ant，groovy，kotlin 这些工具包怎么安装呢，手动下载太麻烦，sdkman 帮你来搞定
<!--more-->

## install sdkman on mac

```bash
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
$ sdk version
  sdkman 5.0.0+51
```

## install software

```bash
sdk install groovy ; sdk install gradle ; sdk install ant ; sdk install maven ; sdk install kotlin ; sdk install scala
```

## change kotlin(software) version

### 展示默认的 kotlin 版本 （show the default kotlin version）

```bash
sdk list kotlin

============================================================
Available Kotlin Versions
============================================================
 > * 1.2.50               1.1.4                1.0.6
     1.2.41               1.1.3-2              1.0.5-2
     1.2.40               1.1.3                1.0.5
     1.2.31               1.1.2-5              1.0.4
     1.2.30               1.1.2-2              1.0.3
     1.2.21               1.1.2                1.0.2
     1.2.20               1.1.1                1.0.1-2
     1.2.10               1.1-beta2            1.0.1-1
     1.2.0                1.1-beta             1.0.1
     1.1.61               1.1-RC               1.0.0
     1.1.60               1.1-M04
     1.1.51               1.1-M02
     1.1.50               1.1-M01
     1.1.4-3              1.1
     1.1.4-2              1.0.7

============================================================
+ - local version
* - installed
> - currently in use
============================================================
```

## whereis the software

```bash
ls ~/.sdkman/candidates
  gradle groovy java
```

## 参考
1. https://sdkman.io/install
2. https://sdkman.io/usage
3. http://imushan.com/2016/03/04/java/framework/SDKMAN%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/
