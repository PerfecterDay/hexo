---
title: mysql高可用-复制
date: 2019-08-11 16:15:00
tags: mysql
category: sql
---

### 复制原理
复制是 mysql 数据库提供的一种高可用高性能解决方案，分为3个步骤：
1. 主服务器把数据更改（写）记录到二进制日志文件中，
2. 从服务器把主服务器的二进制日志恢复到自己的中继日志中
3. 从服务器重做中继日志中的日志，把更改应用到本地数据库上，以达到数据的最终一致性。

![mysql 复制原理](/pics/mysql-replacation.jpg)

使用 `show master status` 或 `show slave status` 可以分别查看主/从服务器上的复制状态。
