---
title: "HashMap详解"
date: 2020-09-17T11:52:23+08:00
author: 小叽
categories: ["Java"]
tags: ["Java"]
draft: false
---

本文解析HashMap的源码中的一些常见的问题，以及JDK1.7和JDK1.8中的差异。
    
<!--more-->

# HashMap简介
1. HashMap是基于哈希表的Map接口的实现。此实现提供所有可选的映射操作，并允许使用null值和null键。
2. JDK1.7中采用：数组+链表，JDK1.8中采用：数组+链表+红黑树(当链表长度超过8时自动转化为红黑树)。
3. 除了非同步和允许使用null之外，HashMap类与Hashtable大致相同。  
PS: Hashtable是一个上古的版本了，现在基本上不会使用到他。如果需要一个线程安全的HashMap可以使用[ConcurrentHashMap](https://blog.csdn.net/zlfprogram/article/details/77524326)，其采用分段锁进行同步不允许有空值(JDK1.7)。

# 容量
## 默认的初始容量以及负载因子

默认的初始容量：
````java
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
````

默认的负载因子：
````java
    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
````
在new一个新的HashMap时可以通过构造`public HashMap(int initialCapacity, float loadFactor)`指定负载因子以及初始容量。
并且HashMap中元素的个数达到initialCapacity*loadFactor时就会调用resize()方法扩容，是以2的倍数扩容，
这样保证了哈希桶的数量必须是2的幂次方，最大存储的数量为1 << 30，这样就有一些问题。
1. 为什么默认的负载因子为0.75f  
    - 加载因子过高，例如为1，虽然减少了空间开销，提高了空间利用率，但同时也增加了查询时间成本。  
    - 加载因子过低，例如0.5，虽然可以减少查询时间成本，但是空间利用率很低，同时提高了rehash操作的次数。  
    - 在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少rehash操作次数，所以，一般在使用HashMap时建议根据预估值设置初始容量，减少扩容操作。  
    - 选择0.75作为默认的加载因子，完全是时间和空间成本上寻求的一种折衷选择。
2. 如果自定义的容量不是2的幂次方会怎么样  
    - HashMap在创建的时候并不会分配内存，只有在第一次往里面添加的时候才会分配分配空间。
    - 在添加元素之前首先会判断容量是否为2的幂次方，如果不是就会向上取2的幂次方 例如：设置的容量为17，HashMap会自动将容量转化为32。
3. 为什么哈希桶的数量必须是2的幂次方
   - HashMap是根据key的hash值决定key放入到哪个桶(bucket)中，存储位置通过 tab=[(n-1) & hash] 公式计算得出。
   - 因为n永远是2的次幂，所以n-1的二进制表示永远都是尾端以连续1的形式表示(00001111，00000011)  
     当(n-1)和hash做与运算时，就是保证每个桶的数据尽可能的分布均匀减少Hash碰撞。
   - 例如：1000 & Hash 这样只有第一位有可能为1，也就是说数据只可能存在第一个桶里面。
   
#JDK1.7和JDK1.8中的主要差异
## HashCode计算
JDK1.7 HashCode计算(一大串骚操作)
````java
// 通过一系列移位操作与异或操作获得元素的哈希值。 JDK 8 中已不再使用该方法
final int hash(Object k) {
        int h = hashSeed;
    	// 如果哈希种子存在，并且进行哈希的元素的String类型
        if (0 != h && k instanceof String) {
            // 就让String使用另一种hash算法(减少DoS攻击)
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
}
````

JDK1.8 HashCode计算(低16位与高16位取异或运算)
````java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
````

## 针对碰撞采用的措施
### JDK1.7
JDK1.7的HashMap采用数组加动态链表的经典方式实现HashMap。在发生Hash碰撞时，所有Hash值相同的数据都会被丢到同一个桶里面。对我们来说链表的插入和删除都是o(1),
但是查找的时间复杂度确实o(n),如果大量相同hash值的数据插入那么HashMap就会退化成链表。拖慢程序的执行，甚至会引发[DoS攻击](https://baike.baidu.com/item/dos%E6%94%BB%E5%87%BB) 。 
为了解决这一问题JDK1.7采取了先判断他是不是字符串，如果是为他专门写了一个hash值的算法。

### JDK1.8
JDK1.7的HashMap采用数组加动态链表加红黑树的方式实现HashMap。  
当一个桶(bucket)中的链表多个8个链表并且哈希表中桶的个数大于64时，才会真正进行让其转化为红黑树，若桶中元素小于等于6时，
树结构还原成链表形式。红黑树是一种近乎平衡的二叉树可以保证查找的时间复度为对数复杂度。  
ps:为什么多余8个时候链表会自动转化为红黑树  
    链表数量出现的概率符合0.5倍的泊松分布，出现大于8个链表的概率小于一千万分之一。  
    源码中的注释写的很清楚，因为树节点所占空间是普通节点的两倍，所以只有当节点足够多的时候，才会使用树节点。也就是说，节点少的时候，
    尽管时间复杂度上，红黑树比链表好一点，
    但是红黑树所占空间比较大，综合考虑，认为只能在节点太多的时候，红黑树占空间大这一劣势不太明显的时候，才会舍弃链表，使用红黑树。
    
### 存在问题
因为HashMap本身是线程不安全的，所以在多线程环境下，可能会发生死锁问题。
这种问题主要原因是HashMap是无序的，存储的顺序和取出的顺序不一致。在多线程的环境下容易成环，形成死循环。  
虽然1.8中保持了顺序，只是有效的减少了出现的概率，并不能完全的解决，所有HashMap在多线程情况还是不能放心的使用。    
参考：[疫苗：Java HashMap的死循环](https://coolshell.cn/articles/9606.html)

## JDK1.8 引入新的API
由于JDK1.8引入了函数式编程的概念，紧接着在HashMap中就添加了和函数式绑定的新的API。例如：foreach()方法。

# Hash碰撞解决方法
原文地址：[https://blog.csdn.net/zeb_perfect/article/details/52574915](https://blog.csdn.net/zeb_perfect/article/details/52574915)  
## 1. 开放地址法
开放地执法有一个公式:Hi=(H(key)+di) MOD m i=1,2,…,k(k<=m-1)
其中，m为哈希表的表长。di 是产生冲突的时候的增量序列。如果di值可能为1,2,3,…m-1，称线性探测再散列。
如果di取1，则每次冲突之后，向后移动1个位置.如果di取值可能为1,-1,2,-2,4,-4,9,-9,16,-16,…k*k,-k*k(k<=m/2)，称二次探测再散列。
如果di取值可能为伪随机数列。称伪随机探测再散列。 
## 2. 再哈希法
当发生冲突时，使用第二个、第三个、哈希函数计算地址，直到无冲突时。缺点：计算时间增加。
比如上面第一次按照姓首字母进行哈希，如果产生冲突可以按照姓字母首字母第二位进行哈希，再冲突，第三位，直到不冲突为止
## 3. 链地址法(拉链法)
将所有关键字为同义词的记录存储在同一线性链表中。如下：
![拉链法.png](/blogPicture/拉链法.png)
## 4. 建立一个公共溢出区
假设哈希函数的值域为[0,m-1],则设向量HashTable[0..m-1]为基本表，另外设立存储空间向量OverTable[0..v]用以存储发生冲突的记录。
