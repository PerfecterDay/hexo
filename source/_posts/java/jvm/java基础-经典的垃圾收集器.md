---
title: java基础-经典的垃圾收集器
date: 2020-06-15 08:17:20
tags: java
category: 
- [java,jvm]
---

<img src="/pics/gc-collector.png" alt="" width=60% height=60%>
图3-6展示了七种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用，图中收集器所处的区域，则表示它是属于新生代收集器抑或是老年代收集。