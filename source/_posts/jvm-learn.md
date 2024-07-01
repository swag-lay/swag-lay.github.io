---
title: jvm-learn
date: 2023-05-23 16:32:08
categories:
- java
- jvm
tags:
- jvm
---

# Java内存区域
## 运行数据区域

JDK1.7

![image-20230523163329596](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20230523163329596.png)

JDK1.8

![image-20230523163752233](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20230523163752233.png)



# JVM回收

JDK1.7

![image-20230523163752233](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/jdk1.7%E5%A0%86%E7%BB%93%E6%9E%84.png)

- 新生代
- 老生代
- 永久代

JDK1.8

![image-20230523164208008](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20230523164208008.png)

- 新生代
- 老生代
- 元空间

## 怎么定位垃圾
- 引用计数法
	存在对A的引用，引用计数器+1，引用失败计数器-1，当计数器为0，证明没有引用，可以回收。
	问题：当需要回收的垃圾之间循环引用，都无法回收了
- 可达性分析方法
	现在更喜欢用这种方式，从根开始检查，在堆中通过引用，如果能找到对象，表示为可达，没有则为不可达，清理不可达的对象（根就是JVM线程栈，本地方法栈，运行时变量池，方法区的静态引用，类对象）main对象启动的内容就是根
![image-20240701160929269](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240701160929269.png)

## 怎么清楚垃圾
### 垃圾回收算法
1. 标记清除

![image-20240701161423629](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240701161423629.png)

2. 标记复制

![image-20240701161526542](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240701161526542.png)


3. 标记压缩

![image-20240701161552105](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240701161552105.png)

## G1垃圾回收器
目前最为主流的垃圾回收器，抛弃了传统从物理上进行的分代算法，采用分区region+逻辑分代的算法
G1垃圾回收器主要将堆内存划分为多个大小相等的区域（称为Region），各个区域根据需要扮演不同的角色，可以被定义为Eden区、Survivor区、Old区和Humongous区（存放大对象，老年代的一部分），采用复制算法针对每个区域进行垃圾回收，同样也支持动态的调整内存大小。同时各个Region不需要连续的存储，颠覆了以往堆内存结构的连续性，具备强大的灵活性，也进一步提高了内存利用率。

![image-20240701161932296](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240701161932296.png)



### 常见面试题

#### 如果判断一个常量为废弃常量

JDK1.7之前运行时常量池逻辑包含字符串常量池存放在方法区，此时hotspot对方法区的实现为永久代

JDK1.7字符串常量池被从方法区拿到了堆中，这里没有提高运行时常量池，也就是说字符串常量池被单独拿到了堆，运行常量池剩下的东西还在方法区，也就是永久代

JDK1.8 hotspot移除永久代用元空间代替，这个时候字符串常量池还在堆中，运行时常量池还在方法区，但方法区的实现从永生代变为了元空间。

假设字符串常量池中存在字符“abc”，如果当前没有任何String对象引用该常量，就说明此常量是废弃常量，如果此时发生内存回收的话而且有必要的话，“abc”就会被清楚

#### 如果判断一个类是无用的类

满足三个条件

1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例
2. 加载该类的classLoader被回收
3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

如果满足上述条件虚拟机可以对类进行回收，但并不是像对象一样不使用了就必然被回收