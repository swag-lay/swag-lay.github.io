---
title: java-threadlocal
date: 2024-07-01 13:43:15
categories:
- java
- threadlocal
tags:
---

# 简介
当多线程访问同一个共享变量出现并发问题，特别是对一个变量进行写入的时候，为了保证线程安全，通常再访问共享变里的时候需要进行额外的同步操作才能保证线程安全性。
ThreadLocal是除了加锁这种同步方法之外的一种保证多线程访问出现不安全的方法，当我们在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量就不会出现线程不安全问题。
提供本地变量，如果创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的副本，在实际多线程操作的时候，操作的是自己本地内存种的变量，从而避免线程安全问题。

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/202012181111507.png)

ThreadLocal不支持子父线程通信，需要使用 InheritableThreadLocal类让子线程从父线程中取得值。

## ThreadLocal的弱引用带来的问题
ThreadLocalMap可以简单的将key视为ThreadLocal，value为代码中放入的值，实际上key并不是ThreadLocal本身，而是一个**弱引用**
首先java中存在四种引用：
- 强引用：永远不会回收
- 软引用：当内存将溢出被回收
- 弱引用：发生gc就回收
- 虚引用：用队列接收对象即将死亡的通知

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/5-GYIVKEun.png)
随着程序的运行，栈中ThreadLocal的强引用会消亡，由于key的弱引用，队中的ThreadLocal对象被回收
对于线程来说，currentThread-ThreadLocalMap-Entry-value-object永远存在强引用，没有被回收，因此造成内存泄漏。
因此ThreadLocal的get，set，remove的时候会清除线程ThreadLocalMap中所有key为null的value。此外如果上述没有生效，建议使用完ThreadLocal变量之后执行remove方法

## InheritableThreadLocal
原理是子线程是通过在父线程中调用new Thread()方法创建子线程，其中init方法会将父线程数据拷贝到子线程。
阿里巴巴的TransmittableThreadLocal

## key到期的清理
- 探测式清理
也就是源码中expungeStaleEntry()方法，遍历散列数组，从开始位置（hash得到的位置）向后探测清理过期数据，将过期数据的 Entry 设置为 null ，沿途中碰到未过期的数据则将此数据rehash后重新在 table 数组中定位，如果定位的位置已经有了数据，则会将未过期的数据放到最靠近此位置的 Entry=null 的桶中（顺序往后延），使 rehash 后的 Entry 数据距离正确的桶的位置更近一些。
从当前节点开始遍历数组，key==null的将entry置为null，key!=null的对当前元素的key重新hash分配位置，若重新分配的位置上有元素就往后顺延。
```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}

```
- 启发式清理
启发式清理需要接收两个参数：
	探测式清理后返回的数字下标
	数组总长度
根据源码可以看出，启动式清理会从传入的下标 i 处，向后遍历。如果发现过期的Entry则再次触发探测式清理，并重置 n。这个n是用来控制 do while 循环的跳出条件。如果遍历过程中，连续 m 次没有发现过期的Entry，就可以认为数组中已经没有过期Entry了。
这个 m 的计算是 n >>>= 1 ，你也可以理解成是数组长度的2的几次幂。
例如：数组长度是16，那么2^4=16，也就是连续4次没有过期Entry，即 m = logn/log2(n为数组长度)
从当前节点开始，进行do-while循环检查清理过期key，结束条件是连续n次未发现过期key就跳出循环，n是经过位运算计算得出的，可以简单理解为数组长度的2的多少次幂次
```java
private boolean cleanSomeSlots(int i, int n) {  //探测式清理后返回的数字下标，这里至少保证了Hash冲突的下标至探测式清理后返回的下标这个区间无过期的Entry, n 数组总长度
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {  // 如果发现过期的Entry就在执行一次探测性清理
            n = len;  //重置n
            removed = true;
            i = expungeStaleEntry(i);   //探测性清理
        }
    } while ( (n >>>= 1) != 0);  // 循环条件: m = logn/log2(n为数组长度)
    return removed;
}

```

set() 方法中，遇到key=null的情况会触发一轮 探测式清理 流程
set() 方法最后会执行一次 启发式清理 流程
rehash() 方法中会调用一次 探测式清理 流程
get() 方法中 遇到key过期的时候会触发一次 探测式清理 流程
启发式清理流程中遇到key=null的情况也会触发一次 探测式清理 流程