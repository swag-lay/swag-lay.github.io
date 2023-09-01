---
title: mysql-dml
date: 2023-08-25 18:14:33
categories:
- 数据库
- mysql
tags:

---

# mysql数据库基本操作-dml

dml：data manipulation language，用来对数据库中表的数据记录进行增删改操作

## 添加数据

```mysql
给指定字段添加数据
insert into 表名 (字段名1,字段名2) values (值1,值2,...);

给全部字段添加数据
insert into 表名 values ();

批量添加数据
insert into 表名 () values (),();
insert into 表名 values (),();
```

注意：

- 字段顺序和值的顺序一一对应
- 字符串和日期型数据应该包含在引号中
- 数据大小在字段的规定范围中

## 修改数据

```mysql
update 表名 set 字段名1=值1 [where 条件];
```

## 删除数据

```mysql
delete from 表名 [where 条件];
```

