---
title: mysql-funs
date: 2023-08-28 09:19:42
categories:
- 数据库
tags:
- mysql
---

# mysql中的函数

## 字符串函数

```mysql
concat(s1,s2,...,sn) #字符串拼接
lower(str) 将str转换为小写
upper(str) 将str转换为大写
lpad(str,n,pad) 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度
rpad(str,n,pad) 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度
trim(str) 去掉字符串头部和尾部的空格
substring(str,start,len) 返回从字符串str从start位置起len个长度的字符串
```

## 数值函数

```mysql
ceil(x) 向上取整
floor(x) 向下取整
mod(x,y) 返回x/y的模
rand() 返回0-1内的随机数
round(x,y) 求参数x的四舍五入的值，保留y位小数
```

## 日期函数

```mysql
curdate() 当前日期
curtime() 当前时间
now() 返回当前日期和时间
year(date) 获取指定date的年份
month(date) 获取指定date的月份
day(date) 获取指定date的日期
date_add(date,interval expr type) 返回一个日期/时间值加上一个时间间隔expr后的时间值
datediff(date1,date2) 返回起始时间date1和结束时间date2之间的天数
```

## 流程函数

```mysql
if(value,t,t) 如果value为true，则返回t，否则返回f
ifnull(value1,value2) 如果value1不为空，返回value1，否则返回value2
case when [val1] then [res1] ...else [default] end 如果val1为true，返回res1，否则返回default默认值
case [expr] when [val1] then [res1] ...else [default] end 如果expr的值等于val1，返回res1，否则返回default默认值
```

