---
title: "try catch finally return"
date: 2020-08-26T14:07:45+08:00
draft: false
author: 小叽
---

本文将介绍有return的情况下try catch finally的执行顺序。

<!--more-->

原文地址：[https://blog.csdn.net/ns_code/article/details/17485221](https://blog.csdn.net/ns_code/article/details/17485221)

# 结论
1. 不管有没有出现异常，finally块中代码都会执行。
2. 当try和catch中有return时，finally仍然会执行。
3. finally是在return后面的表达式运算后执行的(此时并没有返回运算后的值，而是先把要返回的值保存起来，
不管finally中的代码怎么样，返回的值都不会改变，仍然是之前保存的值)，所以函数返回值是在finally执行前确定的。
4. finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。

# 概述
在这里看到了try catch finally块中含有return语句时程序执行的几种情况，但其实总结的并不全，而且分析的比较含糊。
但有一点是可以肯定的，finally块中的内容会先于try中的return语句执行，如果finally语句块中也有return语句的话，
那么直接从finally中返回了，这也是不建议在finally中return的原因。下面来看这几种情况。

# 情况分析
情况一：try中有return，finally中没有return
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

情况二：try和finally中均有return

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
分析：try中的return语句同样被拆分了，finally中的return语句先于try中的return语句执行，因而try中的return被”覆盖“掉了，不再执行。

情况三：finally中改变返回值num
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
分析：虽然在finally中改变了返回值num，但因为finally中没有return该num的值，因此在执行完finally中的语句后，
test()函数会得到try中返回的num的值，而try中的num的值依然是程序进入finally代码块前保留下来的值，因此得到的返回值为10。

情况四：将num的值包装在Num类中
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
分析：从结果中可以看出，同样是在finally中改变了返回值num的值，在情况三中，并没有被try中的return返回（test()方法得到的不是100），
但在这里却被try中的return语句返回了。  

对以上情况的分析，需要深入JVM虚拟机中程序执行exection_table中的字节码指令时操作栈的的操作情况，
可以参考[http://www.2cto.com/kf/201010/76754.html](http://www.2cto.com/kf/201010/76754.html)这篇文章，
也可以参考《深入Java虚拟机：JVM高级特性与最佳实践》第6章中对属性表集合的讲解部分。

# 总结
try语句在返回前，将其他所有的操作执行完，保留好要返回的值，而后转入执行finally中的语句，而后分为以下三种情况：
- 如果finally中有return语句，则会将try中的return语句"覆盖"掉，直接执行finally中的return语句，得到返回值，这样便无法得到try之前保留好的返回值。
- 如果finally中没有return语句，也没有改变要返回值，则执行完finally中的语句后，会接着执行try中的return语句，返回之前保留的值。
- 如果finally中没有return语句，但是改变了要返回的值，这里有点类似与引用传递和值传递的区别，分以下两种情况:
    - 如果return的数据是基本数据类型或文本字符串，则在finally中对该基本数据的改变不起作用，try中的return语句依然会返回进入finally块之前保留的值。
    - 如果return的数据是引用数据类型，而在finally中对该引用数据类型的属性值的改变起作用，try中的return语句返回的就是在finally中改变后的该属性的值。
    