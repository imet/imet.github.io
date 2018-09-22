---
title: "linux 内核基础"
date: 2018-09-19T23:13:46+08:00
lastmod: 2018-09-19T23:13:46+08:00
draft: false
keywords:
- linux
- kernel
description: "linux kernel"
tags:
- linux
categories:
- linux
author: "imet"
---

关于linux内核

1. 用户态和内核态，有啥区别 ？
2. 虚拟地址空间怎么划分的 ？

<!--more-->

## 用户空间 vs 内核空间

### 用户空间和内核空间的划分:

下图展示了用户空间和内核空间的划分:

- user space: 应用程序和C库运行在用户空间
- kernal space: 而核心内核和设备驱动程序，以及硬件则常驻内核空间

![linux_kernel_space]( http://p0.img.imet.top/op/20180919234210_linux_kernel_space.png)

### 虚拟地址空间 vs 物理内存

用户空间和内核空间不是物理内存的空间划分，而是抽象的，依据 CPU 的字长来决定空间的大小的。

- 32位CPU 寻址空间: 2的32次方 B = 4GiB
- 64位CPU 寻址空间: 2的64次方 B （实际使用位数是42位或47位寻址空间）

现代的 CPU 基本上都是 64 位，所以最大寻址空间远超目前的物理内存 4GiB, 8GiB, 64GiB 等

![linux_virtual_address_space]( http://p0.img.imet.top/op/20180920002244_virtual_address_space.png)

1. 关于 TASK_SIZE:  IA32 位架构， TASK_SIZE 为 3GiB, 所以内核空间为 1GiB
2. 64位架构，实际上用不了这么多， 可能会造成一定的空洞，参考 arm 64 的架构

![arm64_kernel_space]( http://p0.img.imet.top/op/20180920004223_arm64_kernel_space.png)

所以虚拟地址空间和物理内存空间是独立的

### 地址空间切换

应用程序要打印一段字符:
1. 应用程序在用户空间调用 C 库的 print 函数
2. C 库中 print 函数进行 write() 系统调用
3. 然后 CPU 切换到内核空间执行系统 write() 调用
4. 当 CPU 执行完系统调用后，又切换到用户空间

![kernal_space_user_space_switch]( http://p0.img.imet.top/op/20180920004602_kernel_space_user_space_switch.png)



## 参考
1. 《深入Linux内核架构》
2. 《Linux内核设计与实现》（原书第3版）
3. 《深入理解计算机系统》
4. [Linux 虚拟内存和物理内存的理解][3]
5. [ARM64 Linux 内核虚拟地址空间][4]

[4]: https://www.cnblogs.com/dyllove98/archive/2013/06/12/3132940.html "Linux 虚拟内存和物理内存的理解"
[5]: https://geneblue.github.io/2017/04/02/ARM64%20Linux%20%E5%86%85%E6%A0%B8%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%E7%A9%BA%E9%97%B4/ "ARM64 Linux 内核虚拟地址空间"
