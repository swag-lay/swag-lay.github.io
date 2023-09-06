---
title: sql_trigger
date: 2023-09-05 10:29:54
categories:
- 数据库
- mysql
tags:
---

# 触发器

触发器是与表相关的数据库对象，指在insert/update/delete之前或之后，触发并执行触发器中定义的sql语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。

使用别名old和new来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

创建

```sql
create trigger ...
before/after insert/update/delete
on ... for each row 
begin 
	trigger_stmt;
end;
```

查看

```sql
show triggers;
```

删除

```sql
drop trigger [schema_name]trigger_name;--如果没有指定schema_name，默认为当前数据库
```

