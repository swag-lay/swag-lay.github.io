---
title: sql_optimize
date: 2023-09-01 09:30:42
categories:
- 数据库
- mysql
tags:
---

# SQL优化

## 插入数据

insert插入：批量插入，手动提交事务，主键顺序插入

大批量插入数据：使用load指令进行插入

```mysql
mysql --local-infile -u root -p
set global local_infile=1;
load data local infile '/root/sql1.log' into table ... fields terminated by ',' lines terminated by '\n';
```

## 主键优化

主键乱序插入会发生页分裂 页合并

主键设计原则

​	尽量降低主键的长度

​	插入数据时，尽量选择顺序插入，选择使用auto_increment自增主键

​	尽量不要使用uuid做主键或者是其他自然主键，如身份证号

​	业务操作时，避免对主键的修改

## order by优化

using filesort：通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都是filesort排序

using index：通过有序索引顺序扫描直接返回有序数据，这种情况称为using index，不需要额外排序，操作效率高。

根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法制

尽量使用覆盖索引

多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（asc/desc）

如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size（默认256k）

## group by优化

通过索引

最左前缀法制

## limit优化

优化思路：一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化

## count优化

优化思路：自己计数

效率排序：count(*)约等于count(1)>count(主键id)>count(字段)

## update优化

InnoDB的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则就会从行锁升级为表锁