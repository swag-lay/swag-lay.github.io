---
title: mysql-dql
date: 2023-08-25 18:32:47
categories:
- 数据库
- mysql
tags:

---

# mysql基本操作-dql

dpl：data query language，用来查询数据库中表的记录。

语法：

```mysql
select 
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段列表
having
	分组后条件列表
order by
	排序字段列表
limit
	分页参数
```

- 基本查询

  ```mysql
  select 字段列表 from 表名;
  select * from 表名;
  select 字段1[as 别名]... from 表名;
  select distinct 字段列表 from 表名;#去除重复操作
  ```

- 条件查询（where）

- 聚合函数（count，max，min，avg，sum）

  将一列数据作为一个整体，进行纵向计算（null值不参与聚合函数的运算）

  ```mysql
  select 聚合函数(字段列表) from 表名;
  ```

- 分组查询（group by）

  ```mysql
  select 字段列表 from 表名 [where 条件] group by 分组字段名 [having 分组后过滤条件];
  ```

  where和having区别

  执行时机不同：where是分组之前进行过滤，不满足where条件，不参与分组；而having是分组之后对结果进行过滤。

  判断条件不同：where不能对聚合函数进行判断，而having可以。

- 排序查询（order by）

  ```mysql
  select 字段列表 from 表名 order by 字段1 排序方式,字段2 排序方式;# asc 升序，desc 降序
  ```

- 分页查询（limit）

  ```
  select 字段列表 from 表名 limit 起始索引,查询记录数;
  ```

  

dql执行顺序
from->where->group by->having->select->order by->limit