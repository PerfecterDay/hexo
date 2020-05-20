---
title: Hystrix 总结
date: 2020-05-17  19:15:12
tags: hystrix
category: 
- [java,hystrix]
---

<img src="/pics/hystrix-command-flow-chart.png" alt="">


hystrix 编程的一般流程
1. 构造一个调用依赖服务的 command
   1. 继承 HystrixCommand 类，重写 `R run()` 方法实现调用逻辑
   2. 继承 HystrixObservableCommand  类，重写 `Observable<String> construct()` 方法

2. 调用 command 的执行方法即可
   1. 同步调用 HystrixCommand 类的 `execute()` 方法
   2. 异步调用 HystrixCommand 类的 `Future<R> queue()` 方法

3. 为 command 配置 fallback
   重写 `R getFallback()` 方法即可




https://my.oschina.net/yu120/blog/656199/
https://github.com/Netflix/Hystrix/wiki/How-To-Use
