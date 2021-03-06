---
title: CPU模式及内存寻址
date: 2018-05-30 19:50:07
tags: os
category: os
typora-root-url: ..\..
---
### CPU原理及寄存器
<img src="/pics/cpu.png" width="60%" height="50%">

<img src="/pics/common-regiisters.png" width="100%" height="50%">

### 寻址方式
 ![IA32指令格式](/pics/IA32instructor-format.png)

寻址，址指的是CPU指令中操作数的地址，寻址就是寻找操作数（包括源操作数和目的操作数）的地址，CPU的寻址方式，就是指CPU指令中支持的给出操作数地址的方式。
寻址方式，从大方向来看可以分为: 
1. 寄存器寻址;
2. 立即数寻址;
3. 内存寻址。
   1. 直接寻址
   2. 基址寻址
   3. 变址寻址
   4. 基址变址寻址

#### 寄存器寻址
它是指操作数在寄存器中，直接从寄存器中拿数据就行了，因此只要在指令中直接给出寄存器名即可。
`mov ax, 0x10` 
`add ax, 0x10` 
上述指令中的源操作数就是寄存器寻址

#### 立即数寻址
立即数就是指令中直接给出的常数，指令中的数据就是操作数，不用去寄存器或者内存中寻找操作数。
`mov ax, 0x10` 这条指令中的目的操作数就是寄存器寻址

#### 内存寻址
操作数在内存中的寻址方式称为内存寻址。
##### 直接寻址
直接寻址，就是将直接在操作数中给出的数字作为内存地址，通过中括号的形式告诉CPU，取此地址中的值作为操作数。如:
   mov ax, [0x1234]
   mov ax, [fs:0x5678]
##### 基址寻址
基址寻址，就是在操作数中用 bx 寄存器或bp寄存器作为地址的起始，地址的变化以它为基础。bx寄存器的默认段寄存器是DS，而bp寄存器的默认段寄存器是SS，即bp和sp都是栈 的有效地址。
##### 变址寻址
变址寻址其实和基址寻址类似，只是寄存器由bx、bp换成了si和di。si是指源索引寄存器(source index)，di是指目的索引寄存器(destination index)。两个寄存器的默认段寄存器都是ds。
   mov [di]，ax :将寄存器ax的值存入ds:di指向的内存 
   mov [si+0x1234], ax ;变址中也可以加个偏移量
#### 基址变址寻址
这是基址寻址和变址寻址的结合，即基址寄存器 bx 或 bp 加一个变址寄存器 si 或 di。如:
   mov [bx+di], ax 将 ax 中的值送入以 ds 为段基址，bx+di 为偏移地址的内存
   add [bx+si], ax 将 ax 与[ds: bx+si]处的值相加后存入内存[ds:bx+si]

### 实模式
x86架构的cpu在启动时都是进入的实模式，实模式的特点是：
0. 段式内存访问机制（段基址+段内偏移），操作数一般16位。
1. 16位的寄存器，20位地址线，最大只能访问1M内存。
2. 内存访问方式是段地址*16+偏移地址构成内存的物理地址，且可以通过在程序中修改段基址来访问任意的真实物理内存，并修改。这样用户程序与系统程序一样，拥有至高无上的权利甚至可以修改操作系统的内存。
3. 因为只有16位寄存器，所以一个段最多只能是64KB大小，超过这个大小就要切换段基址来访问更大范围的内存区域。
4. 一次只能运行一个程序。

写一个实模式的程序，你要了解你的程序将会被加载到内存的什么位置，然后将合适的段加载到内存中合适的位置（没有被他人占用），并且初始化各个数据段寄存器的值，以保证其正确地指向各个段基址。恶意程序可以访问任意的物理地址，包括更改操作系统占用内存。

### 保护模式
保护模式的一些特性：
0. 指令寻址方式扩展，操作数扩展到32位。
1. 除段寄存器外，通用寄存器、指令指针寄存器、标志寄存器等都扩展到32位，地址总线和数据总线为32位。
   <img src="/pics/register-extend.png" width="50%" height="50%">

2. 通过段描述符来描述一段内存，包括段基址、段大小、特权级、权限、属性等等。许多描述符组成**全局段描述符表**，存储在内存中。 每个段描述符8个字节。
3. GDTR：48位寄存器保存**GDT全局描述符表在内存中的段基址及大小，前32位为GDT在内存中的起始地址，后16位为GDT的大小。所以段描述符表最多存储2^16/2^3=2^13个段**。
4. 每个程序都可以在全局描述符表中定义自己的段描述符。
5. 保护模式下的段寄存器叫做**段选择子**，0~1位用来存储RPL(请求特权级),第2位代表是在GDT还是在LDT中，3~15位为索引，表示在GDT或者LDT表中的索引。

进入保护模式的步骤：
1. 打开A20地址线(将端口 0x92 的第1位置1)
2. 填写GDT表并将其加载到 GDTR
3. 将CR0寄存器的PE位置1。

#### 保护模式下的内存访问过程：

##### 分段内存访问（段部件功能）
![保护模式下的内存访问](/pics/segment.png)

因为
1. 段描述符（段描述符表）是在内存中，访问内存比较慢
2. 段描述符格式怪异（历史原因），CPU需要额外操作对其进行整合才能根据其中的内容访问内存。
所以，有个**段描述符缓冲寄存器**，根据局部性原理，访问一个断后一段时间内将会继续访问该段，段描述符缓冲寄存器用来缓存段描述符避免重复计算。

##### 特权级
1. CPU分为4个特权级 ring0/1/2/3，现代操作系统一般只用了0和3，不同特权级下的权限不同，比如一些特权指令只能在0下运行。
2. 每个特权级下都有一个独立的栈，所以最多有4个特权级栈，切换特权级时，也要切换栈。所以说**系统调用是需要开销的**(除了栈切换，还有参数传递和上下文保存的开销)

##### 开启分页
开启分页后的内存访问：
<!--![一级页表访存过程](/pics/first-page-table.png) -->
   <img src="/pics/first-page-table.png" width="60%" height="60%">
   <img src="/pics/secondary-page-table.png" width="60%" height="60%">
图中的段部件就是上文中未开启分页时的访存过程。

既然有了一级页表，为什么还 要搞个二级页表呢?理由如下。
1. 一级页表中最多可容纳 1M(2的20次方)个页表项，每个页表项是 4 字节，能map4KB地址空间，如果页表项全满的话， 便是 4MB 大小，而且也标要求是连续的内存空间，很难找到。使用二级页表的话，页目录表4KB，二级页表4KB，只需要4KB即可分配（但是完全4GB的话，也需要4MB一、二级页表来存储，但是可以离散存储）。一个页目录项能 map 4M地址空间。
2. 一级页表中所有页表项必须要提前建好，原因是操作系统要占用 4GB 虚拟地址空间的高 1GB， 用户进程要占用低 3GB。
3. 每个进程都有自己的页表，进程一多，光是页表占用的空间就很可观了。 
归根结底，我们要解决的是:不要一次性地将全部页表项建好，需要时动态创建页表项。
4. 分页存储管理系统是离散分配，页表是连续存储，所以单个页表大小不得超过系统最大连续可分配单元，这个单元就是单个页面的大小。如果一个页表的大小超过了一个页面（假如是两个页面），那么分页存储管理系统只能分配两个离散的页面存储这个页表，可是PTR（页表寄存器）只有页表始址和页表长度，它默认页表是连续的。为了可以检索两个页面大小的页表，需要另外一个页表来索引这两个离散的页表页面，这就是二级页表。当然如果二级页表也超过了单个页面的大小，那么就需要三级页表，以此类推。

启用分页机制:
1. 准备好页目录表及页表。 
2. 将页表地址写入控制寄存器 cr3（页目录基址寄存器PDTR）。 
3. 寄存器 cr0 的 PG 位置 1。

分页后每个进程都会有一个页表与之对应：

![启用分页后的进程](/pics/process_page_table.png)

##### 内存规划
一个用户进程在保护模式下，所有占用的资源（地址空间）就是在分配给它的页表来定义的。基于上述分析：
1. 如果简单来说，所有用户进程都可以分为两个段：代码段和数据段。操作系统加载程序时，可以定义这两个段的DPL为3，且段选择子的RPL也定义为3，这就是为啥用户进程运行在3的原因。且这两个段能被所有的进程共享，因为最终地址是由页表决定的，只要不同进程定义不同的页表，就不会造成地址冲突。
2. 用户进程一般都会使用操作系统提供的功能（系统调用），操作系统的功能是被所有进程共享的，所以在所有进程的页表中，关于系统内核部分的都是一样的。这也就是通常所说的用户4G空间被分成高1G内核空间和3G用户空间的原因。我们通常将内核控制在1G大小，然后同样为内核创建页表，然后将内核页目录表的768~1022（对应高1G空间，1023指向页目录表本身）映射到内核所在的1G地址空间内。在为进程创建页表时，会直接拷贝内核页目录表的项768~1022个页目录表项，这样进程的高1G地址空间就指向内核了。所有进程768~1022页目录项都是相同的。

##### 特权级深入浅出
CPL:(Current Privilege Level):处理器的当前特权级。
DPL:(Descriptor Privilege Level):描述符特权级，即段描述符中指定的该段的特权级。
1. 欲访问一个非代码段（数据段，栈段等）时，数值上只有CPL ≤ 该段的DPL时，才允许访问该数据段。
2. 欲访问一个代码段时，数值上只有CPL = 该段的DPL时，才允许访问该数据段。就是说跳转到的代码的特权级必须和当前特权级相同才允许执行。因为，首先低特权级的代码不能随便跳转到高特权级的代码去执行，其次 ，高特权级代码也不允许跳转到低特权级代码。
由上可知，代码跳转貌似只能平级跳转，那处理器如何实现特权级变化呢？
+ 唯一一种处理器会从高特权降到低特权运行的情况:处理器从中断处理程序中返回到用户态的时候。
+ 处理器只有通过“门结构”才能由低特权级转移到高特权级。
考虑下面的问题：
正常情况下用户所提交的缓冲区的选择子指向用户自己的数据段。说到这估计您也想到了，问题来了，倘若用户程序怀着一颗有非法企图的心，将参数—缓冲区所在数据段的选择子，用内核数据段的选择子代替(用多个选择子测试几次数据便可推测内核数据段)，这样就把内核破坏了。
<!-- ![非法访问](/pics/cheat-rpl.png) -->
   <img src="/pics/cheat-rpl.png" width="80%" height="80%">
这就需要RPL来解决了：
RPL（Request Privilege Level):请求者特权级，保存在选择子的低两位。当用户程序请求操作系统服务，如果需要提交选择子作为参数，为安全起见，操作系统会把选择子中的 RPL 改为用户程序的 CPL，为此，处理器还提供了修改 rpl 的相关指令。
用户程序的 CPL 是不会骗人的，不可能伪造，它起始是由操作系统在加载用户程序时赋予的，记录在段 寄存器 CS 中的低 2 位，就是 RPL 的位置，而 CS 寄存器只能通过 call、jmp、ret、int、sysenter 等指令修改， 即使改的话，用户程序也只能在 3 级特权下折腾，只要用户进程不请求操作系统服务，它的 CPL 是不会变的， 当它申请了系统服务，如果提交了选择子作为参数，选择子中的 RPL 也会被操作系统修改为用户进程的 CPL。 所以，即使用户程序提交了个伪造的选择子也没用，其 RPL 会被操作系统用其 CPL 替换，还其“真身”。
有了RPL，在进行特权检查时要求：
数值上 CPL≥DPL 并且 RPL≤DPL

总结下不通过调用门、直接访问一般数据和代码时的特权检查规则，对于受访者为代码段时: 
+ 如果目标为非一致性代码段，要求:

   数值上 CPL=RPL=目标代码段 DPL
+ 如果目标为一致性代码段，要求:

   数值上(CPL≥目标代码段 DPL && RPL≥目标代码段 DPL)
+ 受访者若为代码，只有在特权级转移时才会被用到，所以有关代码的特权检查都发生在能够改变代码 段寄存器 CS 和指令指针寄存器 EIP 的指令中，即这些指令要么改变 EIP，要么改变 CS 和 EIP。例如 call、 jmp、int、ret、sysexit 等能改变程序执行流的指令。
对于受访者为数据段时:

   数值上(CPL ≤目标数据段 DPL && RPL ≤ 目标数据段 DPL) 
+ 栈段的特权级检查比较特殊，因为在各个特权级下，处理器都要有相应的栈(后面会说到)，也就是说栈的特权等级要和 CPL 相同。所以往段寄存器 SS 中赋予数据段选择子时，处理器要求 CPL 等于栈段 选择子对应的数据段的 DPL，

   即数值上 CPL = RPL = 用作栈的目标数据段 DPL。

##### 调用门
   <img src="/pics/door.png" width="60%" height="60%">

