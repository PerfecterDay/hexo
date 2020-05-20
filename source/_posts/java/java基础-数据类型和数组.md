---
title: java基础-入门，数据类型和数组
date: 2019-03-06  21:11:40
tags: java
category: java
---
## 基本原则
1. java程序源文件的后缀名必须是 .java ,不能是其他文件后缀
2. 通常情况下，java 程序源文件名是任意的，除非源文件中定义了一个 public 类，此时，源文件名必须与该 public 类的类名相同。
3. 一个源文件中可以有多个类，但是因为上面第2条只能有一个 public 的类。
4. 好的建议是一个文件只定义一个类，文件名与类名保持一致。
5. java标识符是区分大小写的。
6. 标识符可以由字母、下划线、美元符号和数字组成，数字不打头。
7. 标识符不能是 java 关键字或保留字。
8. 标识符不能包含空格。

## Java 数据类型
两大类： **基本类型** 和 **引用类型。**

基本类型

![数值类型](/pics/java_numberic_data.png)

基本类型包括 **boolean 类型** 和 **数值类型** 。

boolean 类型只有两个直接量值： true 、 false 。

数值类型包括整数类型和浮点数类型。

**整数类型**包括： byte 、 short 、 int 、 long 、 char 。

**浮点数类型**包括： float 和 double 。

**引用类型**包括类、接口、数组和特殊的 null 。 null 是引用类型的一个直接量。

### 直接量
并不是所有的类型都可以指定直接量，能指定直接量的只有三种类型：基本类型、字符串类型 和 null 引用。具体而言， Java 支持以下8中类型的直接量：
1. int 类型的直接量：在程序中直接给出的整数类型数值，可分为二进制、十进制、八进制和十六进制4种，二进制以 0B 或 0b 开头，八进制以 0 开头，十六进制以 0x 或 0X 开头。
2. long 类型的直接量： 在整型值后边添加L(小l极度不推荐)后就变成了 long 类型的直接量。
3. double 类型的直接量：直接给出一个标准小数形式或者科学计数法形式的浮点数就是 double 类型额直接量。
4. float 类型的直接量：在一个浮点数后加f或F就变成了 float 类型。
5. boolean 类型直接量： true 、false。
6. char 类型直接量： char 类型的直接量有三种形式，分别是单引号括起来的字符、转义字符和 Unicode 值表示的字符。如 'a' 、 '\n' 和 '\u0061'。
7. String 类型直接量： 一个用双引号括起来的字符序列就是 String 类型直接量。
8. null ：可以赋值给任意引用类型。

### 基本类型的类型转换
Java 的7种数值类型可以相互转换，有两种转换方式：自动转换和强制类型转换。

#### 自动类型转换
Java 所有的数值类型变量可以相互转换，Java 系统支持把某种数据类型的值直接赋值给另一种类型的变量，这种方式称为自动类型转换。
1. 当把一个表数范围小类型的变量或数值赋值给另一个表数范围大的变量时，系统可以自动进行类型转换，否则就需要强制类型转换。
![自动类型转换](/pics/auto-change.jpg)
2. 当把任何基本类型的值和字符串进行连接运算时，基本类型的值将自动转换为字符串类型。

#### 强制类型转换
如果希望把上图中箭头右边的类型转换为左边的类型，则必须进行强制类型转换，强制类型转换额语法是：`(TargetType) value` ，强制类型转换的运算符是（ () ）。强制类型转换会造成精度丢失，被称为“缩小转换”。

#### 表达式类型的自动提升
当一个算术表达式中包含多个基本类型的值时，整个算术表达式的数据类型将发生自动提升。Java定义了如下提升规则：
1. 所有的 byte 、 short 和 char 类型将自动被提升为 int 类型。
2. 整个算术表达式的数据类型自动提升到与表达式中最高等级操作数（表数范围最大）同样的类型。

#### 实例分析
```
calss test
{
    public static void main(string[] args)
    {
        short s=1;
        s=s+1;
        s+=1;
        s++;
        system.out.println(s);
    }
}
```
对于这段代码，编译肯定无法通过的，原因是什么？原因就是s=s+1会出错，这是很久之前很常见的一个面试题，这么多年过去了，解答的套路都固定了，让我们看看从java的创造者视角看这个问题是怎么样的？引用资料java语言规范（JLS）

1.JLS（中文版第三版）5.2 赋值转换

 1. 当把表达式的值赋予一个变量时，就会发生赋值转换：必须把表达式的类型转换为变量的类型；
 2. 如果表达式(=号右边部分)是类型byte，short，char，int的常量表达式，则如果变量类型是byte，short，char，并且常量表达式的值在变量的类型中是可以表示的，那么就执行窄化转换（narrowing conversion），如果不能通过赋值环境中允许的转换把表达式的类型转换成变量的类型，那么编译时会报错。这个可以用来**解释s=1，为何将int赋值给short不会报错**。
 3. `s=s+1`为什么会报错？这里我没有看JLS，因为`s=s+1`，左边有变量参与，编译器在无法分析出该变量的值是什么，因为s为变量，其值不确定无法确定s+1是否超出short范围，为了防止进行类型转换时丢失精度，所以编译器直接当成无法确定来处理，报错了事。so，当有变量为byte，short，char时，编译器就是这么干的。需要知道的是在编译期间，编译器只做语法检查，而不会进行计算动作，也就是说编译器不会对`s+1`是否查出s的范围而进行一次计算判断。
 4. `s++`呢？如有必要将`s+1`的和进行窄化转换，即将`s+1`做强制转换`（short）（s+1）`然后赋给s。（JLS中文三版15.15.1）
 5. 最后s+=1，JLS中文三版15.26.2说对于组合运算符形如`E1 op=E2`的组合赋值表达式等价于`E1=（E!）（（E1）op（E2））`，例如`s+=1`等价于`s=(short)(s+1)`。

## 数组
Java数组要求所有数组元素具有相同的类型。Java 数组是一种引用类型。一旦数组的初始化完成，数组在内存中所占的内存空间就被固定下来了，数组的长度将不会改变。Java数组的长度存储在length属性中。

### 数组的初始化
+ 静态初始化：初始化时由程序员显示指定每个元素的初始值，由系统决定长度。
+ 动态初始化：初始化时程序员只指定数组长度，由系统为数组元素分配初始值。

静态初始化： ``arr = {element1,element2,element3....}``; 此外，静态初始化还可以直接在定义时完成： ``type[] arr={element1,element2,element3....}``
动态初始化： ``arr = new int[10]`` 或者 ``type[] arr= new type[length]`` ; 系统根据 type 类型，自动初始化数组。

### foreach访问数组
Java 5之后提供的语法糖，使得访问数组更加方便，不需要使用数组下标索引即可访问数组。

    for(type var: array | collection){
        var //自动访问每个元素
    }
如果想要改变数组中每个元素的值，使用var并不能保证，此处的var是一个局部变量。还是要用 ``arr[index]=value`` 的方式为数组元素赋值。

### Arrays数组工具类
+ `int binarySearch(type[] arr, type key)` : 使用二分查找在数组arr中查找值为key的元素的索引。如果不存在值为key的元素，返回负数。要求arr数组已经按升序排列。
+ `int binarySearch(type[] arr, int from, int end, type key)` : 这个方法与前一个方法类似，但是它只搜索数组中 [from, end] 索引的元素。
+ `tyep[] copyOf(type[] arr, int len)` : 这个方法会把 arr 数组复制成一个长度为 len 的新数组。如果 len 比 arr 的长度小，则只复制 arr 前 len 个元素，如果比 arr 长度大，则后边的元素初始化为0（整型）、0.0（浮点型）、false（布尔型）、null（引用型）。
+ `tyep[] copyOfRange(type[] arr, int from, int to)` :这个方法与前一个方法类似，但这个方法只复制 arr 数组的 [from,to] 部分。
+ `boolean equals(tyape[] a, type[] b)` : 如果 a, b长度相等且每个数组的元素一一相同（==），返回 true。
+ `void fill(tyape[] a, type value)` : 将 a 的所有元素赋值为 value。
+ `void fill(tyape[] a,int from,int end, type value)` : 将a中索引处于[from,end]的元素全部赋值为 value。
+ `void sort(type[] a)` : 将数组 a 排序。
+ `void sort(type[] a,int from,int end)` : 将数组 a 中[from,end]处的元素排序。
+ `String toString(type[] a)` : 将数组转换成字符串，用逗号连接各个元素。

Java 8 中新增的方法：
+ `void parallelPrefix(type[] a, typeBinaryOperator op)` : 利用 op 参数中的计算方法重新计算数组中每个元素的值，op 计算方法包含 left和right两个参数，right指向当前计算元素的索引，left指向right的前一个，第一个元素时，left值为1 。
+ `void parallelPrefix(type[] a,int from, int to, typeBinaryOperator op)` : 与上个方法类似，但仅计算[from，to]索引处的值。
+ `void setAll(type[] a, IntToTypeFunction generator)` : 使用指定生成器 generator 为所有元素赋值，generator控制元素值的生成算法。
+ `void parallelSetAll(type[] a, IntToTypeFunction generator)` : 同上，但是增加了并行能力。
+ `void parallelSort(type[] a)` : 与 sort 方法类似，只是增加了并行能力。
+ `void parallelSort(type[] a, int from,int end)` : 与上面方法类似，只排序[from,end]处的元素排序。
+ `Spliterator<T> spliterator(T[] array)` : 将数组转换成 Spliterator 对象。
+ `Spliterator<T> spliterator(T[] array, int startInclusive, int endExclusive)` : 同上，值转换[startInclusive,endExclusive]的元素。
+ `Stream<T> stream(T[] array)` : 将数组转换成 Stream 。
+ `Stream<T> stream(T[] array，int startInclusive, int endExclusive)` : 同上。

## 语句

### switch 语句
switch 语句后面的表达式类型只支持 byte 、 short 、char 、 int 四中整数类型（不支持 long 类型），String (Java 7开始支持)和枚举类型。
switch 语句会先计算出表达式的值，然后拿着个表达式的值和 case 标签后的值比较，一旦遇到相等的值，程序就开始执行这个 case 标签后的代码，不再判断后边的 case 、 default 标签的条件是否匹配，除非遇到 break 。

### jar 命令详解
jar 是 JDK 自带的工具，依赖于 JDK 的 tools.jar 包。
+ -c  创建新档案
+ -t  列出档案目录
+ -x  从档案中提取指定的 (或所有) 文件
+ -u  更新现有档案
+ -v  在标准输出中生成详细输出
+ -f  指定档案文件名
+ -m  包含指定清单文件中的清单信息
+ -n  创建新档案后执行 Pack200 规范化
+ -e  为捆绑到可执行 jar 文件的独立应用程序指定应用程序入口点
+ -0  仅存储; 不使用任何 ZIP 压缩
+ -P  保留文件名中的前导 '/' (绝对路径) 和 ".." (父目录) 组件
+ -M  不创建条目的清单文件
+ -i  为指定的 jar 文件生成索引信息
+ -C  更改为指定的目录并包含以下文件
示例：
1. jar -cf test.jar test: 将test路径下的全部内容打包成 test.jar
2. jar -cvf test.jar test: 同上，但是会显示打包过程
3. jar -cvfM test.jar test: 同上，不生成清单文件
4. jar -cvfm test.jar manifest.mf test: 同上，使用指定的清单文件
5. jar tvf test.jar: 查看指定 jar 的文件列表的详细信息
6. jar -xvf test.jar: 解压缩指定 jar 文件