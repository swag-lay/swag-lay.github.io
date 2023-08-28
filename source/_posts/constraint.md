---
title: constraint
date: 2023-08-28 12:25:25
categories:
- 数据库
tags:
- mysql
---

# mysql约束

约束是作用于表中字段上的规则，用于限制存储在表中的数据。为了保证数据库中数据的正确，有效性和完整性。

| 约束                   | 描述                                                     | 关键字      |
| ---------------------- | -------------------------------------------------------- | ----------- |
| 非空约束               | 限制该字段的数据不能为null                               | not null    |
| 唯一约束               | 唯一，不重复的                                           | unique      |
| 主键约束               | 唯一标识，要求非空且唯一                                 | primary key |
| 默认                   |                                                          | default     |
| 检查约束（8.0.16之后） | 保证字段值满足某一个条件                                 | check       |
| 外键约束               | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | foreign key |

注意：约束是作用于表中字段上的，可以在创建表/修改表的时候添加约束。

外键约束：

```mysql
create table 表名(
	字段名 数据类型,
    ...
    [constraint] [外键名称] foreign key(外键字段名) references 主表(主表列名)
);
-- 添加外键
alter table 表名 add constraint 外键名称 foreign key(外键字段名) references 主表(主表列名);

-- 删除外键
alter table 表名 drop foreign key 外键名称;
```

删除/更新行为

| 行为        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no action   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。（与restrict一致） |
| restrict    | 。。。 ，首先检查该记录是否有对应外键，如果有则不允许删除/更新。（与no action一致） |
| cascade     | 。。。，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录 |
| set null    | 。。。，首先检查该记录是否有对应外键，如果有则设置子表中该外键为null（这就要求该外键允许曲null） |
| set default | 父表有变更时，子表将外键列设置为一个默认的值（Innodb不支持） |

```mysql
alter table 表名 add constraint 外键名称 foreign key (外键名称) references 主表名（主表字段名） on update cascade on delete cascade;
```

