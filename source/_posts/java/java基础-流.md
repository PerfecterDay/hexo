---
title: Java基础-流
date: 2021-01-22  22:32:18
tags: stream
category: java
typora-root-url: ..\..
---

### Java 流概述
Java 8 中增加了 Stream API，简化了串行或并行的大批量操作。这个 API 提供了两个关键抽象：
1. Stream（流）
   代表数据元素的有限或无限顺序，这些元素可能来自任何位置，常见的来源包括集合、数组、文件、正则表达式模式匹配器、伪随机数生成器，以及其他 Stream 。Stream 中的数据元素可以是对象引用、或者基本类型值。它支持三种基本类型：int、long 和 double 。

2. Strean pipeline（流管道）
   代表这些元素的一个多级计算。一个 Strean pipeline 中包含一个源 Stream ，接着是 0 或者多个中间操作和一个终止操作。每个中间操作都会通过某种方式对 Stream 进行转换，例如将每个元素通过一个函数映射到另一元素，或者过滤掉某些不满足条件的元素。所有的中间操作都会将 Stream 转换成另一个 Stream， 其元素类型可能与输入的 Stream 一样，也可能不同。终止操作会在最后一个中间操作产生的 Stream 上执行一个最终计算，例如将其元素保存到一个集合中，并返回某一个元素，或打印出所有元素等。
   Stream pipeline 通常是 lazy 的：直到调用终止操作时才会开始计算，对于完成终止操作不需要的数据元素，将永远都不会被计算。正是这种 lazy 计算，使无限 Stream 成为可能。没有终止操作的 Stream pipeline 将是一个静默的无操作指令，因此千万不能忘记终止操作。
   Stream API 是流式的：所有的 pipeline 的调用可以链接成一个表达式。事实上，多个 pipeline 也可以链接在一起，成为一个表达式。
   默认情况下，Stream pipeline 是按顺序运行的。要使 pipeline 并发执行，只需在该 pipeline 的任何 Stream 上调用 parallel 方法即可，但是通常不建议这么做。

### Collectors API
Collectors API 又叫收集器，它的作用是将 Stream 的元素合并到单个对象中去，收集器产生的对象一般是一个集合。
将 Stream 的元素集中到一个真正的 Collection 里去的收集器比较简单，它有三个这样的收集器： `toList()`、`toSet()`、`toCollection(collectionFactory)`，它们分别返回一个列表、一个集合和程序员指定的集合类型。`toMap(keyMapper,valueMapper)` 将 Stream 元素集合到 Map 中，keyMapper 是一个将 Stream 元素映射到 Map 中键的函数，而 valueMapper 是将 Stream 元素映射到 Map 中值的函数。