---
title: select_tables
date: 2023-08-28 13:53:25
categories:
- 数据库
tags:
- mysql
---

# mysql多表查询

概述：各个表结构之间存在着多种联系，基本上分为三种：

一对多（多对一）：

​	在多的一方建立外键，指向一的一方的主键（比如部门和员工的关系）

多对多

​	建立第三种中间表，中间表至少包含两个外键，分别关联两方主键（例如学生和课程的关系）

一对一

​	多用于单边拆分，将一张表的基础字段放在一张表中，其他详细字段放在另一张表中，以提升操作效率

​	在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的（unique）

## 多表查询分类

### 连接分类

#### 内连接：

相当于查询A，B交集部分数据

隐式内连接

```mysql
select .. from .. where 条件...;
```

显式内连接

```
select .. from 表1 [inner] join 表2 on 连接条件;
```

#### 外连接：

左外连接：查询左表所有数据，以及两张表交集部分数据

```mysql
select .. from 表1 left [outer] join 表2 on .. #相当于查询表1（左表）的所有数据 包含 表1和表2交集部分的数据
```

右外连接：查询右表所有数据，以及两张表交集部分数据

```mysql
select .. from .. right [outer] join .. on .. #相当于查询表2（右表）的所有数据 包含 表1和表2交集部分的数据
```

#### 自连接：

当前表与自身的连接查询，自连接必须使用表别名。

自连接可以用内连接和外连接。

```mysql
select .. from 表a 别名a join 表a 别名b on 条件 ..;
```

### 子查询

sql语句中嵌套select语句，称为嵌套查询，又称子查询

```mysql
select * from t1 where column1=(select column1 from t2);
```

子查询外部的语句可以是insert/update/delete/select中的任意一个

根据子查询结果不同，分为：

标量子查询（子查询结果为单个值）

列子查询（子查询结果为一列）

​	常用操作符：in，not in，any，some ，all

行子查询（子查询结果为一行）

​	常用操作符：=，<>，in，not in

表子查询（子查询结果为多行多列）

​	常用操作符：in

根据子查询位置，分为：where之后，from之后，select之后

## 联合查询

对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集

union all会把所有的查询直接联合，union会去重

```mysql
select .. from ..
union [all]
select .. from ..;
```

