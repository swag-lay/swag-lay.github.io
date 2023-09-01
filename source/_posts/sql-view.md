---
title: sql_view
date: 2023-09-01 12:59:44
categories:
- 数据库
- mysql
tags:
---

# SQL视图

介绍：视图是一种虚拟存储的表，视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。通俗来讲，视图只保存了查询的sql逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在了创建这条sql查询语句上。

## 创建

```mysql
create [or replace] view 视图名称[(列名列表)] as select语句 [with[cascaded | local] check option]
```

## 查询

```mysql
查看创建视图语句：show create view ...;
查看视图数据：select * from ...;#表怎么查视图怎么查
```

## 修改

```mysql
create [or replace] view 视图名称[(列名列表)] as select语句 [with[cascaded | local] check option]
alter view 视图名称[(列名列表)] as select语句 [with[cascaded | local] check option]
```

## 删除

```mysql
drop view [if exists] 视图名称[(列名列表)]
```

## 视图的检查选项

使用with check option子句创建视图时，mysql会通过视图检查正在修改的每一行，例如插入，更新，删除，以使其符合视图的定义。mysql允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。为了确定检查的范围，mysql提供了两个选项：cascaded和local，默认值cascaded。

local 不会把上一个视图也加上一致性检查

## 视图的更新

要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图中包含以下任何一项，则视图不可更新

​	聚合函数或窗口函数

​	distinct

​	group by

​	having

​	union或者union all