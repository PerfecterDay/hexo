---
title: java基础-范型
date: 2019-03-04  22:08:24
tags: 范型
category: java
---
原文：https://blog.csdn.net/u011240877/article/details/53545041

泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

类型参数的意义是告诉编译器这个集合中要存放实例的类型，从而在添加其他类型时做出提示，在编译时就为类型安全做了保证。

这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

    /**
    * <header>
    *      Description: 泛型类
    * </header>
    * <p>
    *      Author: shixinzhang
    */
    public class GenericClass<F> {
        private F mContent;

        public GenericClass(F content){
            mContent = content;
        }

        /**
        * 泛型方法
        * @return
        */
        public F getContent() {
            return mContent;
        }

        public void setContent(F content) {
            mContent = content;
        }

        /**
        * 泛型接口
        * @param <T>
        */
        public interface GenericInterface<T>{
            void doSomething(T t);
        }
    }

## 泛型类
泛型类和普通类的区别就是**类名后有类型参数列表 <E>**，既然叫“列表”了，当然这里的类型参数可以有多个，比如 `public class HashMap<K, V>` ，参数名称由开发者决定。

类名中声明参数类型后，内部成员、方法就可以使用这个参数类型，比如上面的 GenericClass<F> 就是一个泛型类，它在类名后声明了类型 F，它的成员、方法就可以使用 F 表示成员类型、方法参数/返回值都是 F 类型。

泛型类最常见的用途就是作为容纳不同类型数据的容器类，比如 Java 集合容器类。

## 泛型接口
和泛型类一样，泛型接口在接口名后添加类型参数，比如上面的 GenericInterface<T>，接口声明类型后，接口方法就可以直接使用这个类型。

实现类在实现泛型接口时需要指明具体的参数类型，不然默认类型是 Object，这就失去了泛型接口的意义。

未指明类型的实现类，默认是 `Object` 类型：

    public class Generic implements GenericInterface{

        @Override
        public void doSomething(Object o) {
            //...
        }
    }
指明了类型的实现：

    public class Generic implements GenericInterface<String>{
        @Override
        public void doSomething(String s) {
            //...
        }
    }
泛型接口比较实用的使用场景就是用作策略模式的公共策略，比如 Java 解惑：Comparable 和 Comparator 的区别 中介绍的 Comparator，它就是一个泛型接口：

    public interface Comparator<T> {

        public int compare(T lhs, T rhs);

        public boolean equals(Object object);
    }
泛型接口定义基本的规则，然后作为引用传递给客户端，这样在运行时就能传入不同的策略实现类。

## 泛型方法
泛型方法是指使用泛型的方法，如果它所在的类是个泛型类，那就很简单了，直接使用类声明的参数。

如果一个方法所在的类不是泛型类，或者他想要处理不同于泛型类声明类型的数据，那它就需要自己声明类型，举个例子:

    /**
    * 传统的方法，会有 unchecked ... raw type 的警告
    * @param s1
    * @param s2
    * @return
    */
    public Set union(Set s1, Set s2){
        Set result = new HashSet(s1);
        result.addAll(s2);
        return result;
    }

    /**
    * 泛型方法，介于方法修饰符和返回值之间的称作 类型参数列表 <A,V,F,E...> (可以有多个)
    *      类型参数列表 指定参数、返回值中泛型参数的类型范围，命名惯例与泛型相同
    * @param s1
    * @param s2
    * @param <E>
    * @return
    */
    public <E> Set<E> union2(Set<E> s1, Set<E> s2){
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }





## 泛型的通配符

1. <?> 无限制通配符
2. <? extends E> extends 关键字声明了类型的上界，表示参数化的类型可能是所指定的类型，或者是此类型的子类
3. <? super E> super 关键字声明了类型的下界，表示参数化的类型可能是指定的类型，或者是此类型的父类

## 泛型的类型擦除
Java 中的泛型和 C++ 中的模板有一个很大的不同：

+ C++ 中模板的实例化会为每一种类型都产生一套不同的代码，这就是所谓的代码膨胀。
+ Java 中并不会产生这个问题。虚拟机中并没有泛型类型对象，所有的对象都是普通类。
（摘自：http://blog.csdn.net/fw0124/article/details/42295463）

在 Java 中，泛型是 Java 编译器的概念，用泛型编写的 Java 程序和普通的 Java 程序基本相同，只是多了一些参数化的类型同时少了一些类型转换。

实际上泛型程序也是首先被转化成一般的、不带泛型的 Java 程序后再进行处理的，编译器自动完成了从 Generic Java 到普通 Java 的翻译，Java 虚拟机运行时对泛型基本一无所知。

当编译器对带有泛型的java代码进行编译时，它会去执行类型检查和类型推断，然后生成普通的不带泛型的字节码，这种普通的字节码可以被一般的 Java 虚拟机接收并执行，这在就叫做 类型擦除（type erasure）。

实际上无论你是否使用泛型，集合框架中存放对象的数据类型都是 Object，这一点不仅仅从源码中可以看到，通过反射也可以看到。

    List<String> strings = new ArrayList<>();
    List<Integer> integers = new ArrayList<>();
    System.out.println(strings.getClass() == integers.getClass());//true

## 擦除的实现原理

一直有个疑问，Java 编译器在编译期间擦除了泛型的信息，那运行中怎么保证添加、取出的类型就是擦除前声明的呢？
从这篇文章了解到，原来泛型也只是一个语法糖，大概意思就是：

Java 编辑器会将泛型代码中的类型完全擦除，使其变成原始类型。当然，这时的代码类型和我们想要的还有距离，接着 Java 编译器会在这些代码中加入类型转换，将原始类型转换成想要的类型。这些操作都是编译器后台进行，可以保证类型安全总之泛型就是一个语法糖，它运行时没有存储任何类型信息。