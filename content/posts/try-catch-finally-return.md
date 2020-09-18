---
title: "try-catch-finally-return"
date: 2020-08-30T18:07:45+08:00
draft: false
categories: ["Java"]
tags: ["Java"]
author: 小叽
---

本文将介绍有 return 的情况下 try-catch-finally 的执行顺序。

<!--more-->

原文地址：[https://blog.csdn.net/ns_code/article/details/17485221](https://blog.csdn.net/ns_code/article/details/17485221)

# 结论
1. 不管有没有出现异常，finally 块中代码都会执行。
2. 当 try 和 catch 中有 return 时，finally 仍然会执行。
3. finally 是在 return 后面的表达式运算后执行的(此时并没有返回运算后的值，而是先把要返回的值保存起来，
不管 finally 中的代码怎么样，返回的值都不会改变，仍然是之前保存的值)，所以函数返回值是在 finally 执行前确定的。
4. finally 中最好不要包含 return，否则程序会提前退出，返回值不是 try 或 catch 中保存的返回值。

# 概述
在这里看到了 try-catch-finally 块中含有 return 语句时程序执行的几种情况，但其实总结的并不全，而且分析的比较含糊。
但有一点是可以肯定的，finally 块中的内容会先于 try 中的 return 语句执行，如果 finally 语句块中也有 return 语句的话，
那么直接从 finally 中返回了，这也是不建议在 finally 中 return 的原因。下面来看这几种情况。

# 情况分析
情况一：try 中有 return，finally 中没有 return
````java
public class TryTest {
    public static void main(String[] args) {
        System.out.println(test());
    }

    private static int test() {
        int num = 10;
        try {
            System.out.println("try");
            return num += 80;
        } catch (Exception e) {
            System.out.println("error");
        } finally {
            if (num > 20) {
                System.out.println("num>20 : " + num);
            }
            System.out.println("finally");
        }
        return num;
    }
}
````
输出结果：  
````
try
num>20 : 90
finally
90
````
分析：显然“return num += 80”被拆分成了“num = num+80”和“return num”两个语句，线执行try中的“num = num+80”语句，
将其保存起来，在try中的”return num“执行前，先将finally中的语句执行完，而后再将90返回。

情况二：try 和 finally 中均有return

````java
public class TryTest {
    public static void main(String[] args) {
        System.out.println(test());
    }

    private static int test() {
        int num = 10;
        try {
            System.out.println("try");
            return num += 80;
        } catch (Exception e) {
            System.out.println("error");
        } finally {
            if (num > 20) {
                System.out.println("num>20 : " + num);
            }
            System.out.println("finally");
            num = 100;
            return num;
        }
    }
}
````
输出结果：  
````
try
num>20 : 90
finally
100
````
分析：try 中的 return 语句同样被拆分了，finally 中的 return 语句先于 try 中的 return 语句执行，因而 try 中的 return 被“覆盖”掉了，不再执行。

情况三：finally 中改变返回值num
````java
public class TryTest {
    public static void main(String[] args) {
        System.out.println(test());
    }

    private static int test() {
        int num = 10;
        try {
            System.out.println("try");
            return num;
        } catch (Exception e) {
            System.out.println("error");
        } finally {
            if (num > 20) {
                System.out.println("num>20 : " + num);
            }
            System.out.println("finally");
            num = 100;
        }
        return num;
    }
}
````
输出结果：  
````
try
finally
10
````
分析：虽然在 finally 中改变了返回值 num，但因为 finally 中没有 return 该 num 的值，因此在执行完 finally 中的语句后，
test()函数会得到 try 中返回的 num 的值，而 try 中的 num 的值依然是程序进入 finally 代码块前保留下来的值，因此得到的返回值为10。

情况四：将 num 的值包装在 Num 类中
````java
public class TryTest {
    public static void main(String[] args) {
        System.out.println(test().num);
    }

    private static Num test() {
        Num number = new Num();
        try {
            System.out.println("try");
            return number;
        } catch (Exception e) {
            System.out.println("error");
        } finally {
            if (number.num > 20) {
                System.out.println("number.num>20 : " + number.num);
            }
            System.out.println("finally");
            number.num = 100;
        }
        return number;
    }
}

class Num {
    public int num = 10;
}
````
输出结果：  
````
try
finally
100
````
分析：从结果中可以看出，同样是在 finally 中改变了返回值 num 的值，在情况三中，并没有被 try 中的 return 返回(test()方法得到的不是100)，
但在这里却被 try 中的 return 语句返回了。  

对以上情况的分析，需要深入JVM虚拟机中程序执行exection_table中的字节码指令时操作栈的的操作情况，
可以参考[http://www.2cto.com/kf/201010/76754.html](http://www.2cto.com/kf/201010/76754.html)这篇文章，
也可以参考《深入Java虚拟机：JVM高级特性与最佳实践》第6章中对属性表集合的讲解部分。

# 总结
try语句在返回前，将其他所有的操作执行完，保留好要返回的值，而后转入执行 finally 中的语句，而后分为以下三种情况：
- 如果 finally 中有 return 语句，则会将 try 中的 return 语句"覆盖"掉，直接执行 finally 中的 return 语句，得到返回值，这样便无法得到 try 之前保留好的返回值。
- 如果 finally 中没有 return 语句，也没有改变要返回值，则执行完 finally 中的语句后，会接着执行 try 中的 return 语句，返回之前保留的值。
- 如果 finally 中没有 return 语句，但是改变了要返回的值，这里有点类似与引用传递和值传递的区别，分以下两种情况:
    - 如果 return 的数据是基本数据类型或文本字符串，则在 finally 中对该基本数据的改变不起作用，try 中的 return 语句依然会返回进入 finally 块之前保留的值。
    - 如果 return 的数据是引用数据类型，而在 finally 中对该引用数据类型的属性值的改变起作用，try 中的 return 语句返回的就是在 finally 中改变后的该属性的值。
    