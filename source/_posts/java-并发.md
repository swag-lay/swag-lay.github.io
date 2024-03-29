---
title: java_并发
date: 2023-12-22 15:02:42
categories:
- java
- 并发编程
tags:
---
# 并发编程线程基础
进程是系统进行资源分配的基本单位
线程是进程的一个执行路径，线程是cpu分配的基本单位

线程创建的三种方式和优缺点
1. 继承Thread类并重写run方法
* 优点：在run()方法内直接获取当前线程直接使用this就可以，不用使用Thread.currentThread()方法
* 缺点：不支持多继承，无法继承其他类
	任务与代码没有分离，多个线程执行一样的任务时需要多份任务代码
	没有返回值
2. 实现Runnable接口的run方法
* 优点：解决继承Thread类单继承的局限性
	降低线程对象和线程任务之间的耦合性，增强程序可扩展性
	将线程单独进行对象的封装，符合面向对象的思想
* 缺点：没有返回值
3. 实现Callable接口（使用FutureTask方式）
* 优点：有返回值
	解决继承Thread类单继承的局限性
	降低线程对象和线程任务的耦合性，增强程序可扩展性
	将线程单独进行对象的封装，更符合面向对象的思想
* 缺点：编程和理解稍复杂
## 线程的通知和等待
## 线程上下文切换
cpu资源的分配采用了时间片轮转的策略，也就是给每个线程分配一个时间片，线程在时间片内占用cpu执行任务。人气线程使用完时间片后，就会处于就绪状态并让出cpu让其他线程占用，这就是上下文切换
切换的时机：

1. 当前线程的cpu时间片使用完处于就绪状态时
2. 当前线程被其他线程中断时
## 线程死锁
1. 互斥
2. 请求保存
3. 不可剥夺
4. 环路等待
synchronized 是 Java 中的关键字，用于控制多线程访问共享资源时的同步。它可以保证同一时间只有一个线程执行特定代码区块，从而避免并发问题如数据不一致或竞态条件。原子性内置锁，线程的执行代码在进行synchronized代码块前会自动获取内部锁，这时候其他线程访问该同步代码块时会被阻塞挂起。拿到内部锁的线程会在正常退出同步代码块或者抛出异常后或者在同步块内调用了该内置锁资源的wait系列方法时释放该内置锁。
## 守护线程和用户线程
守护线程和用户线程的区别：当最后一个非守护线程结束时，jvm会正常退出，而不管当前时候有守护线程
ThreadLocal：线程的局部变量，ThreadLocal的变量只有当前自身线程可以访问，别的线程都访问不了

## 保证线程安全的思路
1. 使用没有共享资源的模型
2. 使用共享资源只读不写的模型：
	1. 不需要写共享资源的模型
	2. 使用不可变对象
3. 直面线程安全
	1. 保证原子性
	2. 保证顺序性
	3. 保证可见性

## 为什么程序计数器，虚拟机栈和本地方法栈是线程私有的呢？为什么堆和方法区是线程共享的呢？
程序计数器私有主要是为了**线程切换后能恢复到正确的执行位置**。
需要注意，如果执行的是native方法，那么程序计数器记录的是undefined地址，只有执行的是java代码时程序计数器记录的才是下一条指令的地址
虚拟机栈：为虚拟机执行java（也就是字节码）服务。
本地方法栈：为虚拟机使用到的native方法服务。
因此，为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。

堆是进程中最大的一块内存，主要用于存放新创建的对象，方法区主要用于存放已被加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。

java创建线程的方式：无论多少种方式，本质都是通过new Thread().start.