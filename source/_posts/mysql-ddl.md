---
title: mysql-ddl
date: 2023-08-25 16:41:00
categories:
- 数据库
tags:
- Mysql
---

# mysql数据库基本操作-ddl

## 对数据库的常用操作

| 功能             | sql                                                  |
| ---------------- | ---------------------------------------------------- |
| 查看所有的数据库 | show databases;                                      |
| 创建数据库       | create database [if not exists] mydb [charset=utf8]; |
| 切换数据库       | use mydb;                                            |
| 删除数据库       | drop database [if exists] mydb;                      |

## 对表结构的常用操作

### 创建表

```mysql
create table 表名(
    字段名1 类型[(宽度)] [约束条件] [comment '字段说明'],
    字段名2 类型[(宽度)] [约束条件] [comment '字段说明'],
    ...
)[表的一些设置];
```

### 数据类型

常用

| tinyint                  | 1byte   | (-128,127)               | (0,255)             |
| ------------------------ | ------- | ------------------------ | ------------------- |
| smallint                 | 2       | (-32768,32767)           | (0,65535)           |
| mediumint                | 3       | (-8388608,8388607)       | (0,16777215)        |
| int/integer              | 4       | (-2147483648,2147483647) | (0,4294967295)      |
| bigint                   | 8       |                          |                     |
| float                    | 4       |                          |                     |
| double                   | 8       |                          |                     |
| decimal                  |         | 依赖与M和D的值           | 依赖与M和D的值      |
| vachar                   | 0-65532 | 变长字符串               |                     |
| char                     | 0-255   | 定长字符串               |                     |
| date（日期值）           | 3       | 1000-01-01至9999-12-31   | YYYY-MM-DD          |
| time（时间值或持续时间） | 3       | -838:59:59至838:59:59    | HH:MM:SS            |
| datetime                 | 8       |                          | YYYY-MM-DD HH:MM:SS |

### 修改表

添加字段

```mysql
alter table 表名 add 字段名1 类型[(宽度)] [约束条件] [comment '字段说明'];
```

修改数据类型

```mysql
alter table 表名 modify 字段名 新数据类型（长度）;
```

修改字段名和字段类型

```mysql
alter table 表名 change 旧字段名 新字段名 类型（长度） [comment '字段说明'] [约束];
```

删除字段

```mysql
alter table 表名 drop 字段名;
```

修改表名

```mysql
alter table 表名 rename to 表名;
```

删除表

```mysql
drop table [if exists] 表名;
```

删除指定表，并重新创建该表

```mysql
truncate table 表名
```

