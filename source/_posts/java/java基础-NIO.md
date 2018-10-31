---
title: java-NIO
date: 2018-11-03  08:43:24
tags: NIO
category: java
---
主要转自：https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html

## 流与块的比较
面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的 I/O 通常相当慢。

一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

## 通道
Channel是一个对象，可以通过它读取和写入数据。拿 NIO 与原来的 I/O 做个比较，通道就像是流。

正如前面提到的，所有数据都通过 Buffer 对象来处理。您永远不会将字节直接写入通道中，相反，您是将数据写入包含一个或者多个字节的缓冲区。同样，您不会直接从通道中读取字节，而是将数据从通道读入缓冲区，再从缓冲区获取这个字节。

通道与流的不同之处在于通道是双向的。而流只是在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)， 而 通道 可以用于读、写或者同时用于读写。

## 缓冲区
Buffer 是一个对象， 它包含一些要写入或者刚读出的数据。 在 NIO 中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，您将数据直接写入或者将数据直接读到 Stream 对象中。

在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问 NIO 中的数据，您都是将它放到缓冲区中。

缓冲区实质上是一个数组。通常它是一个字节数组，但是也可以使用其他种类的数组。但是一个缓冲区不 仅仅 是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

### 缓冲区的状态变量
可以用三个值指定缓冲区在任意时刻的状态：

+ capacity
+ position
+ limit
这三个变量一起可以跟踪缓冲区的状态和它所包含的数据。
#### Capacity
缓冲区的 capacity 表明可以储存在缓冲区中的最大数据容量。实际上，它指定了底层数组的大小 ― 或者至少是指定了准许我们使用的底层数组的容量。

limit 决不能大于 capacity。

#### position
position 变量跟踪已经写了多少数据。更准确地说，它指定了下一个字节将放到数组的哪一个元素中。因此，如果您从通道中读三个字节到缓冲区中，那么缓冲区的 position 将会设置为3，指向数组中第四个元素。

同样，在写入通道时，您是从缓冲区中获取数据。 position 值跟踪从缓冲区中获取了多少数据。更准确地说，它指定下一个字节来自数组的哪一个元素。因此如果从缓冲区写了5个字节到通道中，那么缓冲区的 position 将被设置为5，指向数组的第六个元素。

#### limit
Limit
limit 变量表明还有多少数据需要取出(在从缓冲区写入通道时)limit等于缓冲区中数据的数量，或者还有多少空间可以放入数据(在从通道读入缓冲区时)，limit应该等于capacity。

position 总是小于或者等于 limit。

#### flip方法
现在我们要将数据写到输出通道中。在这之前，我们必须调用 flip() 方法。这个方法做两件非常重要的事：

+ 它将 limit 设置为当前 position。
+ 它将 position 设置为 0。

#### clear方法
从通道读入数据到缓冲区之前，这个方法重设缓冲区以便接收更多的字节。 Clear 做两件非常重要的事情：

+ 它将 limit 设置为与 capacity 相同。
+ 它设置 position 为 0。

#### get() 方法
ByteBuffer 类中有四个 get() 方法：

+ byte get();
+ ByteBuffer get( byte dst[] );
+ ByteBuffer get( byte dst[], int offset, int length );
+ byte get( int index );
第一个方法获取单个字节。第二和第三个方法将一组字节读到一个数组中。第四个方法从缓冲区中的特定位置获取字节。那些返回 ByteBuffer 的方法只是返回调用它们的缓冲区的 this 值。

此外，我们认为前三个 get() 方法是相对的，而最后一个方法是绝对的。 相对 意味着 get() 操作服从 limit 和 position 值 ― 更明确地说，字节是从当前 position 读取的，而 position 在 get 之后会增加。另一方面，一个 绝对 方法会忽略 limit 和 position 值，也不会影响它们。事实上，它完全绕过了缓冲区的统计方法。

上面列出的方法对应于 ByteBuffer 类。其他类有等价的 get() 方法，这些方法除了不是处理字节外，其它方面是是完全一样的，它们处理的是与该缓冲区类相适应的类型。

#### put()方法
ByteBuffer 类中有五个 put() 方法：

+ ByteBuffer put( byte b );
+ ByteBuffer put( byte src[] );
+ ByteBuffer put( byte src[], int offset, int length );
+ ByteBuffer put( ByteBuffer src );
+ ByteBuffer put( int index, byte b );
第一个方法 写入（put） 单个字节。第二和第三个方法写入来自一个数组的一组字节。第四个方法将数据从一个给定的源 ByteBuffer 写入这个 ByteBuffer。第五个方法将字节写入缓冲区中特定的 位置 。那些返回 ByteBuffer 的方法只是返回调用它们的缓冲区的 this 值。

与 get() 方法一样，我们将把 put() 方法划分为 相对 或者 绝对 的。前四个方法是相对的，而第五个方法是绝对的。

上面显示的方法对应于 ByteBuffer 类。其他类有等价的 put() 方法，这些方法除了不是处理字节之外，其它方面是完全一样的。它们处理的是与该缓冲区类相适应的类型。

#### 缓冲区分配和包装
在能够读和写之前，必须有一个缓冲区。要创建缓冲区;
1. 分配它。我们使用静态方法 allocate() 来分配缓冲区：

    ByteBuffer buffer = ByteBuffer.allocate( 1024 );
2. 还可以将一个现有的数组转换为缓冲区，如下所示：

    byte array[] = new byte[1024];
    ByteBuffer buffer = ByteBuffer.wrap( array );

### Selector
Selector 就是您注册对各种 I/O 事件的兴趣的地方，而且当那些事件发生时，就是这个对象告诉您所发生的事件。
我们关心某个channel是否发生了读写或者Accept事件，就把这个channel和相应的事件通过channel的register方法注册到selector对象上，这样selector会在这些channel上发生了你感兴趣的事件时通知你。

所以，我们需要做的第一件事就是创建一个 Selector：
    
    Selector selector = Selector.open();


