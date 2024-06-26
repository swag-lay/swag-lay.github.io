---
title: jvm-class-loading-process
date: 2023-05-24 15:56:37
categories:
- java
- jvm
tags:
- 类加载
---

# 类加载

## 类的生命周期

![image-20240522165900485](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240522165900485.png)



## 类加载过程

上图中的加载->连接->初始化，连接过程又分为三步：验证->准备->解析

- 加载

  1. 通过类名获取此类的二进制字节流

  2. 将字节流代表的静态存储结构转换为方法区的运行时数据结构

  3. 在内存中生成一个代表该类的class对象，作为方法区这些数据的访问入口
  

每个java类都有一个引用指向加载它的ClassLoader。但是数组类不是通过ClassLoader创建的，而是JVM在需要的时候自动创建的，数组类通过getClassLoader()方法获取ClassLoader的时候和该数组的元素类型的ClassLoader是一致的。非数组类的加载阶段是可控性最强的阶段，这一步我们可以去完成还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的loadClass()）。加载阶段和连接阶段的部分动作是交叉进行的，加载阶段尚未结束，连接阶段就有可能开始了。

```java
class Class<T> {
  ...
  private final ClassLoader classLoader;
  @CallerSensitive
  public ClassLoader getClassLoader() {
     //...
  }
  ...
}
```

### 类加载器加载规则

根据需要动态加载。对于已经加载的类会被加载在ClassLoader中。

```java
public abstract class ClassLoader {
  ...
  private final ClassLoader parent;
  // 由这个类加载器加载的类。
  private final Vector<Class<?>> classes = new Vector<>();
  // 由VM调用，用此类加载器记录每个已加载类。
  void addClass(Class<?> c) {
        classes.addElement(c);
   }
  ...
}
```

### 类加载器总结

![image-20240522165923428](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240522165923428.png)

除了BootstrapClassLoader是JVM本身的一部分，其他所有的类加载器都是在JVM外部实现的，并且全部继承自ClassLoader抽象类。这与做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。



- 连接

  1. 验证

     确保Class文件的字节流中包含的信息符合所有约束条件，保证信息被当作代码运行后不会危害虚拟机自身的安全。

     ![image-20240522165940785](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240522165940785.png)

  2. 准备
  
     正式为类变量分配内存并设置类变量初始值的阶段
  
  3. 解析
  
     虚拟机将常量池内的符号引用替换为直接引用的过程

- 初始化

  是执行初始化方法<clinit>()方法的过程，是类加载的最后一步，这一步JVM才开始真正执行类中定义的Java程序代码（字节码）

  ## 类卸载

  卸载类即该类的Class对象被GC

  三个条件：

  1. 该类的而所有实例对象被GC，也就是是堆中不存在该类的实例对象
  2. 该类在任何地方被引用
  3. 该类的类加载器的实例被GC

  因此，在JVM的生命周期中，由jvm自带的类加载器加载的类是不会被卸载的，但是由我们自定义的类加载器加载的类是可能被卸载的。
