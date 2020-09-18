---
title: "genericity"
date: 2020-09-05T11:05:18+08:00
author: 小叽
categories: ["Java"]
tags: ["Java"]
draft: false
---

Java语言中泛型是“假”泛型！！！

<!--more-->

# 简介
泛型是程序设计语言的一种特性。允许程序员在强类型程序设计语言中编写代码时定义一些可变部分，那些部分在使用前必须作出指明。
各种程序设计语言和其编译器、运行环境对泛型的支持均不一样。
将类型参数化以达到代码复用提高软件开发工作效率的一种数据类型。泛型类是引用类型，是堆对象，主要是引入了类型参数这个概念。

# 没有泛型的Java
其实，在远古时代 (java5之前)Java 是没有泛型的。  
在没有泛型的年代我们只能通过自定义的“StringList”类来实现类型安全。  
自定义 StringList 实现：

````java
public class Test {
    public static void main(String[] args) {
        StringList stringList = new StringList();
        stringList.add("1");
        String result = stringList.get(0);
        System.out.println(result);
    }

    static class StringList {
        private List list;

        public StringList() {
            list = new ArrayList();
        }

        public void add(String element) {
            list.add(element);
        }

        public String get(int index) {
            return list.get(index).toString();
        }
    }
}
````
可想而知，如果需要一个 Integer 类型的就要把上面的代码复制一般定义一个IntegerList，
需要一个 String[] 类型的就要定义一个 StringArrList，需要。。。。。。  
这样想想就很可怕。。。如果不这样做你根本不知道取出来的元素是什么类型的，这样对于后面的操作可能会很头疼。

# 泛型出现
泛型程序设计（Generic Programming）意味着编写的代码可以被很多不同类型的对象所重用。  

自从泛型出现我们就可以用省⼒的方法编写类型安全的代码，并且可以舒服的使用 `List<String>`、`List<Integer>` 等等。

## 向后兼容
但是这样仍然存在问题，为了保证向后兼容性，只有两条路可以选择：
1. 擦除 -> Java的选择
2. 搞一套全新的API -> C#的选择

## 擦除带来的问题
擦除导致 Java 中的泛型只存在编译期间，在运行期间完全不保留
所以说 Java 泛型是“假”泛型、伪泛型，是编译时期泛型。简单来讲，在运行时(在编译后的字节码文件中)，所有的泛型都是object。  
可以通过反射绕过编译器的检测：
````
//通过反射绕过泛型检查，即泛型擦除
Class<ArrayList> listClass = ArrayList.class;
Method addMethod = listClass.getMethod("add", Object.class);
//编译器不再报错，程序也可以正常执行
addMethod.invoke(list, "a");
````
注意：`List<String>` 不是 `List<Object>` 的子类。

## 泛型的限定符
1. ? extends 要求泛型是某种类型及其⼦子类型 
2. ? super 要求泛型是某种类型及其⽗父类型 
3. Collections.sort

## 泛型的绑定
泛型方法示例：
````
public static <T extends Comparable<T>> boolean compare(T a, T b, T c) {
    return a.compareTo(b) <= 0 && b.compareTo(c) <= 0;
}
````

调用泛型方法：
````
public static void main(String[] args) {
    System.out.println(compare(1, 2, 3));
    System.out.println(compare(1L, 2L, 3L));
    System.out.println(compare(1d, 2d, 3d));
}
````
