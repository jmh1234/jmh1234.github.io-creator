---
title: "Java内存区域与内存溢出"
date: 2020-09-18T15:26:08+08:00
categories: ["Java", "JVM"]
tags: ["Java", "JVM"]
draft: false
---

Java 和 C++ 之间有一堵由内存分配和垃圾收集技术所围成的“高墙”，墙外的想进去，墙内的人想出去  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;《深入理解Java虚拟机》


<!--more-->

# 概述

对于 C、C++ 的程序开发人员来说，在内存管理领域，他们既是拥有最高权力的“皇帝”又是从事最基础劳动的“劳动人民”——既拥有对象的“所有权”，
又担负着每一个对象生命开始到终结的维护责任。  
对于 Java 程序员来说，在虚拟机自动内存管理机制的帮助下，不再需要为每一个 new 操作去写配对的 delete/free 代码，不容易出现内存泄露和内存溢出问题，
由虚拟机管理内存这一切看起来都挺美好。不过也正是 Java 程序员把内存控制的权利交给了Java虚拟机，使得排查问题变得很困难。

# 运行时数据区
Java虚拟机在执行Java程序的过程中会把他管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建时间和销毁时间，有的区域随着虚拟机进程启动二存在，
有的区域则依赖用户线程的启动和结束而建立和销毁。
## 程序计数器

程序计数器(Program Counter Register) 是一块较小的内存空间，它可以看做时当前线程所执行的字节码的行数指示器。在虚拟机的概念模型里，
字节码解释器工作时就是通过改变这个计数器的值来选取下一条需执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能操作都依赖这个计数器来完成。
每条线程都需要有一个独立的程序计数器，各条线程之间的程序计数器互不影响，独立存储。这类内存称为“线程私有”的内存

## Java虚拟机栈
与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期和线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧(Stack Frame)
用于存储局部变量表、操作数栈、动态链接、方法出入口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程
