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

### 局部变量表
局部变量表存储了编译期可知的各种基本数据类型(boolean、byte、char、short、int、float、long、double)、对象引用(reference类型，它不等同于对象本身，
可能是指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置)和returnAddress类型(指向一条字节码指令的地址)。  
局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要再栈中分配多大的局部变量空间是完全确定的，在方法的运行期间是不糊改变局部变量表的大小。

### 操作数栈
每一个独立的栈帧中除了包含局部变量表以外，还包含一个后进先出（LIFO）的操作数栈，其通过标准的入栈和出栈操作来完成一次数据访问。
每一个操作数栈都会拥有一个明确的栈深度用于存储数值，一个32bit的数值可以用一个单位的栈深度来存储，而2个单位的栈深度则可以保存一个64bit的数值。
当然操作数栈所需的容量大小在编译期就可以被完全确定下来，并保存在方法的Code属性中。  
简单来说，操作数栈就是JVM执行引擎中的一个工作区，当一个方法被调用时，一个新的栈帧也会随之被创建，但这个时候栈帧中的操作数栈却是空的，
只有方法在执行的过程中，才会有各种各样的字节码指令往操作数栈中执行入栈和出栈操作。
比如一个方法内部需执行一个简单的加法运算时，首先需要从操作数栈中将需要执行运算的两个数值出栈，待运算执行完成后，再将运算结果入栈。

## 本地方法栈
本地方法栈(Native Method Stack) 与虚拟机栈发挥的作用是非常相似的，他们之间的区别不过是虚拟机栈为虚拟机执行Java方法(也就是字节码)服务，
而本地方法栈则为虚拟机使用到的 Native 方法服务。一般情况会把虚拟机栈和本地方法栈合二为一，统称为栈。 

