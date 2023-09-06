---
title: sql_stored
date: 2023-09-04 10:38:27
categories:
- 数据库
- mysql
tags:
---

# 存储过程

介绍：事先经过编译并存储在数据库中的一段sql语句的集合，调用存储过程并可以简化应用开发任意的许多工作。减少数据在数据库和应用数据库之间的传输。

数据库sql语言层面的代码封装和重用。

特点：

封装，复用。

可以接收参数，也可以返回数据

减少网络交互，效率提升

创建：

```sql
create procedure 存储过程名称([参数列表])
begin
	--sql语句
end;
```

调用：

```sql
call 名称([参数]);
```

查看

```sql
select * from information_schema.routines_schema='xxx';--查询指定数据库的存储过程及状态信息
show create procedure 存储过程名称;--查询某个存储过程的定义
```

删除

```sql
drop procedure [if exists] 存储过程名称;
```

注意：在命令行中，执行创建存储过程的sql时，需要通过关键字delimiter指定sql语句的结束符



## 变量

系统变量是mysql服务器提供的，不是用户定义的，属于服务器层面。分为全局变量（global），会话变量（session）。

查看系统变量

```sql
show [session|global] variables; --查看所有系统变量
show [session|global] variables like '...'; --可以通过like模糊匹配方式查找变量
select @@[session|global]系统变量名; --查看指定变量的值
```

设置系统变量

```mysql
set [session|global] 系统变量名=值;
set @@[session|global]系统变量名=值;
```

注意：如果没有指定session/global，默认是session，会话变量

mysql服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在/etc/my.cnf中配置



用户定义变量

是用户根据需要自己定义的变量，用户变量不用提取声明，在用的时候直接用“@变量名”使用就可以。其作用域为当前连接。

赋值：

```sql
set @var_name=expr [,@var_name=expr]...;
set @var_name:=expr [,@var_name:=expr]...;
select @var_name:=expr [,@var_name:=expr]...;
select 字段名 into @var_name from 表名;
```

使用

```sql
select @var_name;
```

注意：用户定义的变量无需对其进行声明或初始化，只不过获取到的值为null。



局部变量

是根据需要定义的在局部生效的变量，访问之前，需要declare声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的begin...end块。

声明

```sql
declare 变量名 变量类型[...];
```

赋值

```sql
set 变量=值;
set 变量名:=值;
select 字段名 into 变量名 from 表名...;
```



if

```mysql
if ... then 
	...
elseif ... then 
	...
else 
	...
end if;
```

参数

| 类型  | 含义                                     | 备注 |
| ----- | ---------------------------------------- | ---- |
| in    | 该类参数作为输入，也就是需要调用时传入值 | 默认 |
| out   | 该类参数作为输出，作为返回值             |      |
| inout | 即可以作为输入参数，也可以作为输出参数   |      |

```sql
create procedure 存储过程名称([in/out/inout 参数名 参数类型])
begin

end;
```

case

```mysql
case ...
	when ... then ...
	..
	[else ...]
end case;
```

```mysql
case 
	when ... then ...
	..
	[else ...]
end case;
```

while

```mysql
while ... do
	...
end while;
```

repeat

```mysql
repeat 
	..
	until ...
end repeat;
```

loop

```mysql
[begin_lable:] loop
	...
end loop[end_label];

leave label;--退出指定标记的循环体
iterate label;--直接进入下一次循环
```

游标：

用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环的处理。

声明

```mysql
declare ... cursor for ...;
```

打开

```mysql
open ...;
```

获取游标记录

```mysql
fetch ... into 变量[,变量];
```

关闭游标

```mysql
close ...;
```

条件处理程序

```sql
declare handler_action  handler for condition_value[,condition_value]... statement;

handler_action
	continue:继续执行当前程序
	exit:终止执行当前程序
condition_value
	sqlstate sqlstate_value:状态码，如02000
	sqlwarning:所有以01开头的sqlstate代码的简写
	not found:所有以02开头的sqlstate代码的简写
	sqlexception:所有没有被sqlwarning或not found捕获的sqlstate代码的简写
```

# 存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是in类型

```mysql
create function 存储函数名称([参数列表])
returns type [characteristic ...]
begin
	...
	return ...;
end;
```

characteristic说明

​	deterministic：相同的输入参数总数产生相同的结果

​	no sql：不包含sql语句

​	reads sql data：包含读取数据的语句，但不包含写入数据的语句

