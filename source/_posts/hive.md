---
title: hive
date: 2023-09-12 15:18:18
categories:
- 大数据开发
- hive
tags:
---

# Hive

apache hive是一款分布式sql计算的工具，其主要功能是：

将sql语句翻译成mapreduce程序运行

使用hive的好处

操作接口采用类sql语法，快速开发

底层执行mapreduce，可以完成分布式海量数据的sql处理

![image-20230912162030090](..\images\分布式sql计算.png)

## 元数据管理

构建分布式sql计算

元数据管理功能，即：

​	数据位置

​	数据结构

​	对数据进行描述，进行记录

sql解析器

​	sql分析

​	sql到mapreduce程序的转换

​	提交mapreduce程序运行并收集执行结果

## hive组件

用户接口

包括CLI，jdbc/odbc，webgui。

cli：shell命令行

hive中的thrift服务器运行外部客户端通过网络和hive进行交互，类似于jdbc或odbc协议

webgui是通过浏览器访问hive

![image-20230912162901255](..\images\hive架构图.png)

## hive数据库基本语法

基本和mysql一样