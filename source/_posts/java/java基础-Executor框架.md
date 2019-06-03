---
title: java基础-Executor框架
date: 2019-08-25  19:30:28
tags: java          
category: java
---

#### Executor 框架的结构
Executor 框架主要由3大部分组成：
1. 任务。包括被执行任务需要实现的接口： Runnable 接口和 Callable 接口。
2. 任务的执行。包括任务执行机制的核心接口 Executor ，以及继承自 Executor 的 ExecutorService 接口。Executor框架有两个关键类实现了 ExecutorService 接口（ThreadPoolExecutor 和 ScheduledThreadPoolExecutor）。
3. 异步计算的结果。包括 Future 接口和实现 Future 接口的 FutureTask 类。

![Executor框架的类与接口](/pics/executor-framework.jpg)
![Executor框架的使用示意图 ](/pics/how-to-use-executor.jpg)