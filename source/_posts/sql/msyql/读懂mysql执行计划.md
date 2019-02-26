---
title: 读懂mysql执行计划
date: 2018-06-19 15:22:28
tags: mysql
category: sql
---

在MySQL使用 **explain** 关键字来查看SQL的执行计划，如下所示：
![mysql执行计划](/pics/mysql-explain1.png)

+ id：表示查询中 select 操作表的顺序,按顺序从大到依次执行
+ select_type:表示选择的类型,可选值有:   SIMPLE(简单的), PRIMARY(最外层) ，SUBQUERY(子查询中的第一个select查询) , derived(衍生) , union , unionresult 
+ type:显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是： system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL ,一般来说，得保证查询至少达到range级别，最好能达到ref。确定SQL是否走索引,走了什么索引,也就可以通过该属性查看了。
    * ALL: 扫描全表
    * index: 扫描全部索引树
    * range: 扫描部分索引，索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行，常见于between、<、>等的查询
    * ref: 非唯一性索引扫描，返回匹配某个单独值的所有行。常见于使用非唯一索引即唯一索引的非唯一前缀进行的查找
    * eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
    * const, system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量。system是const类型的特例，当查询的表只有一行的情况下， 使用system。
    * NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引。
+ table :表示该语句查询的表。
+ possible_keys:顾名思义,该属性给出了该查询语句，可能走的索引,(如某些字段上索引的名字)这里提供的只是参考,而不是实际走的索引,也就导致会有 possible_keys 不为 null , key 为空的现象。
+ key :显示MySQL实际使用的索引,其中就包括主键索引(PRIMARY),或者自建索引的名字.
+ key_len :显示MySQL决定使用的键长度。表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。如果键是NULL，长度就是NULL。显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。
+ ref :连接匹配条件,如果走主键索引的话,该值为: const, 全表扫描的话,为 null 值
+ rows :扫描行数,也就是说，需要扫描多少行,才能获取目标行,一般情况下会大于返回行数。通常情况下, rows 越小,效率越高,也就有大部分SQL优化，都是在减少这个值的大小。注意:理想情况下扫描的行数与实际返回行数理论上是一致的,但这种情况及其少,如关联查询,扫描的行数就会比返回行数大很多
+ Extra：这个属性非常重要,该属性中包括执行SQL时的真实情况信息,如上面所述,使用到的是 using where ，表示使用 where 筛选得到的值,常用的有:
"Using temporary" : 使用临时表, "using filesort": 使用文件排序
