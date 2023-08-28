---
title: mysql-dcl
date: 2023-08-25 18:32:55
categories:
- 数据库
tags:
- mysql
---

# 数据库基本操作-dcl

dcl：data control language，数据控制语言，用来管理数据库用户，控制数据库的访问权限。

## 管理用户

查询用户

```mysql
use mysql；select * from user；
```

创建用户

```mysql
create user '用户名'@'主机名' identified by '密码'；
```

修改用户密码

```mysql
alter user '用户名'@'主机名' identified with mysql_native_password by '新密码';
```

删除用户

```mysql
drop user '用户名'@'主机名';
```

注意：主机名可以用%

## 权限控制

查询权限

```msyql
show grants for '用户名'@'主机名';
```

授予权限

```mysql
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';
```

撤销权限

```mysql
revoke 权限列表 on 数据库名.表名 to '用户名'@'主机名';
```

