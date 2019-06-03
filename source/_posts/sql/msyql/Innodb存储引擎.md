---
title: mysql innodb 存储引擎
date: 2019-09-18 21:11:23
tags: mysql
category: sql
---


### Innodb 体系架构

#### 后台线程
1. Master Thread  
Master Thread 是核心线程，主要负责将缓冲池中的缓存异步刷新到磁盘，保证数据额一致性，包括脏页的刷新、合并插入缓冲、undo 页的回收等。

2. IO Thread  
Innodb 存储引擎大量使用异步 IO 来处理 IO请求，这样做可以提高数据库的性能。在 1.0 版本之前，有4个IO 线程，分别是 read、write、insert buffer 和 log IO thread 。从1.0.x 版本开始 read 和 write 的线程数增大到4个，并且可以通过 `innodb_read_io_threads` 和 `innodb_write_io_threads` 参数进行设置。可以通过 `show engine innodb status` 来查看 innodb 引擎的运行状态。

3. Purge Thread  
Purge Thread 用来回收已经提交的事务的 undo log页。 可以在配置文件中配置 `innodb_purge_threads=4` 来配置回收线程的数量。

4. Page Cleanner Thread  
该线程的主要作用是将脏页的刷新操作放入到本线程中来，减轻 Master thread 的工作，提高系统性能。

#### 内存缓冲池

`innodb_buffer_pool_instances`