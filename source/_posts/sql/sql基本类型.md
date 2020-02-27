---
title: sql基本类型
date: 2018-06-14 20:00:00
tags: sql   
category: sql
---

sql标准支持多种数据类型,包括：

+ **char(n)**:**固定**长度的字符串，用户指定长度n，也可以使用全称character(n);
+ **varvhar(n)**:**可变长度**的字符串用户指定**最大长度n**，等价于全称character varying(n);
+ **int**:整数类型，等价于全称integer;
+ **smallint**:小整数类型;
+ **numeric(p,d)**:定点数，精度由用户指定。这个数有p位数字(包括一位符号位)，其中d位数字在小数点右边。所以一个numberic(3,1)可以精确表示44.5,但是不能精确表示444.5和0.32;
+ **real,double precision**:浮点数与双精度浮点数，精度与机器有关;
+ **float(n)**:精度至少为n位的浮点数。
每种类型都可能包含一个称作**空值**的特殊值。

**char(10)**类型的字段存储“Aou”三个字符时，不足10位的部分会补上7个空格来填充，而**varchar(10)**存储相同值时只会占用三个字符的空间，因此更推荐使用**varchar(n)**数据类型。

Mysql 的数据类型：
![Mysql 数据类型](/pics/mysql-types.jpg)
