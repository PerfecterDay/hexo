---
title: Hystrix 总结
date: 2020-05-17  19:15:12
tags: hystrix
category: 
- [springboot,hystrix]
---

### 命令模式
现在有个需求，有一个类 RemoteContrller 需要调用若干类的一些方法，这些类和方法的名字都是不一样的，而且未来可能需要支持更多类的方法调用。如何设计这个 RemoteContrller 类，使得能够支持后边调用类的扩展？具体来说， RemoteContrller 可能会涉及到下边这些类的方法调用：
<img src="/pics/vendor_class.png" alt="">
设计一个 Command 接口，RemoteController 针对 Command 编程，只要调用它的 execute 方法就好了；不用去关心具体的调用逻辑。对应的，对上图中每个类的方法调用都要封装成一个 Command 对象，里边包含着上述类的引用及具体方法调用的实现逻辑，我们可以在此处实现方法调用的扩展和自定义。
<img src="/pics/command_pattern.png" alt="">

### Hystrix
Hystrix 就是使用了上述的命令模式。微服务的场景下，对下游服务的调用实际上就可以看成命令模式中对方法的调用，我们可以使用命令模式，对下游服务调用的方法进行封装和额外处理，比如调用失败次数统计，服务降级，服务熔断...

#### Hystrix 相关概念及配置
1. 服务降级（fallback)：当服务器压力剧增的情况下，根据当前业务情况及流量对一些服务和页面有策略的降级，以此释放服务器资源以保证核心任务的正常运行。下面四种情况会触发服务降级:
   + 非HystrixBadRequestException异常：当抛出HystrixBadRequestException时，调用程序可以捕获异常，没有触发getFallback()，而其他异常则会触发getFallback()，调用程序将获得getFallback()的返回
   + run()/construct()运行超时：如使用无限while循环或sleep模拟超时，会触发了getFallback()
   + 熔断器启动：假如我们配置10s内请求数大于3个时就启动熔断器，请求错误率大于80%时就熔断，然后for循环发起请求，当请求符合熔断条件时将触发getFallback()。更多熔断策略见下文
   + 线程池/信号量已满：假如配置线程池数目为3，然后先用一个for循环执行queue()，触发的run()sleep 2s，然后再用第2个for循环执行execute()，发现所有execute()都触发了fallback，这是因为第1个for的线程还在sleep，占用着线程池所有线程，导致第2个for的所有命令都无法获取到线程
2. 服务熔断：当下游的服务因为某种原因突然变得不可用或响应过慢，上游服务为了保证自己整体服务的可用性，不再继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。
3. 隔离策略： hystrix提供了两种隔离策略：线程池隔离和信号量隔离。hystrix默认采用线程池隔离。
   1. 线程池隔离：不同服务通过使用不同线程池，彼此间将不受影响，达到隔离效果。以demo为例，我们通过andThreadPoolKey配置使用命名为ThreadPoolTest的线程池，实现与其他命名的线程池天然隔离，如果不配置andThreadPoolKey则使用withGroupKey配置来命名线程池
   2. 信号量隔离：线程隔离会带来线程开销，有些场景（比如无网络请求场景）可能会因为用开销换隔离得不偿失，为此hystrix提供了信号量隔离，当服务的并发数大于信号量阈值时将进入fallback。以demo为例，通过withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)配置为信号量隔离，通过withExecutionIsolationSemaphoreMaxConcurrentRequests配置执行并发数不能大于3，由于信号量隔离下无论调用哪种命令执行方法，hystrix都不会创建新线程执行run()/construct()，所以调用程序需要自己创建多个线程来模拟并发调用execute()，最后看到一旦并发线程>3，后续请求都进入fallback

```
 public CommandHelloWorld(String name) {
   super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"))
            .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("HelloWorldPool"))
            .andThreadPoolPropertiesDefaults(	// 配置线程池
                		HystrixThreadPoolProperties.Setter()
                		.withCoreSize(2)	// 配置线程池里的线程数，设置足够多线程，以防未熔断却打满 threadpool
                )
                .andCommandPropertiesDefaults(	// 配置熔断器
                		HystrixCommandProperties.Setter()
                		.withCircuitBreakerEnabled(true)
                		.withCircuitBreakerRequestVolumeThreshold(3)
                		.withCircuitBreakerErrorThresholdPercentage(80)
//                		.withCircuitBreakerForceOpen(true)	// 置为true时，所有请求都将被拒绝，直接到fallback
//                		.withCircuitBreakerForceClosed(true)	// 置为true时，将忽略错误
//                		.withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)	// 信号量隔离
//                		.withExecutionTimeoutInMilliseconds(5000)
            );
   this.name = name;
}
```
+ HystrixCommandKey :每个 HystrixCommandKey 代表一个依赖抽象,相同的依赖要使用相同的 HystrixCommandKey 名称。依赖隔离的根本就是对相同 HystrixCommandKey 的依赖做隔离.
+ HystrixCommandGroupKey: Hystrix 使用 HystrixCommandGroupKey 来为相同的 Command 分组.
+ HystrixThreadPoolKey： 一个 HystrixCommand 与一个 HystrixThreadPool 相关联（HystrixCommand 需要 HystrixThreadPool 来执行），HystrixThreadPoolKey 就是用来代表关联的 HystrixThreadPool 的，如果没有显示指定，会用 HystrixCommandGroupKey 代表。
+ circuitBreakerEnabled：是否支持断路器熔断。
+ circuitBreakerErrorThresholdPercentage: 配置当错误率达到多少时，打开断路器。

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
