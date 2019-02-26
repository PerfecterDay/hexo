---
title: java基础-数组
date: 2019-03-09  11:53:00
tags: java数组
category: java
---

## 数组
Java 数组是一种引用类型。一旦数组的初始化完成，数组在内存中所占的内存空间就被固定下来了，数组的长度将不会改变。

## 数组的初始化
+ 静态初始化：初始化时由程序员显示指定每个元素的初始值，由系统决定长度。
+ 动态初始化：初始化时程序员只指定数组长度，由系统为数组元素分配初始值。

静态初始化： ``arr = {element1,element2,element3....}``; 此外，静态初始化还可以直接在定义时完成： ``type[] arr={element1,element2,element3....}``
动态初始化： ``arr = new int[10]`` 或者 ``type[] arr= new type[length]`` ; 系统根据 type 类型，自动初始化数组。

## foreach
Java 5之后提供的语法糖，使得访问数组更加方便，不需要使用数组下标索引即可访问数组。

    for(type var: array | collection){
        var //自动访问每个元素
    }
如果想要改变数组中每个元素的值，使用var并不能保证，此处的var是一个局部变量。还是要用 ``arr[index]=value`` 的方式为数组元素赋值。

## Arrays工具类
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
