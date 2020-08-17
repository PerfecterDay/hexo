---
title: java基础-反射
date: 2018-05-10  20:08:35
tags: 反射
category: java
typora-root-url: ..\..
---

# 反射
---------------
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取类的信息以及动态调用对象的方法的功能称为java语言的反射机制。

要想解剖一个类,必须先要获取到该类的字节码文件对象，进而使用的Class类中的方法解剖类。所以先要获取到一个字节码文件对应的Class类型的对象。

## Class对象
反射把一个java类字节码加载到内存并构造成一个Class类型的对象，并将类中的属性、方法等各种成分映射成相应的Java对象。

一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把各个组成部分映射成一个个对象。（其实：一个类中这些成员方法、构造方法等在Java中都有一个专门的类来描述）。如图是类的正常加载过程：反射的原理在于class对象。
熟悉一下加载：Class对象的由来是将class字节码文件读入内存，并为之创建一个Class对象。
![字节码文件加载实例化Class对象](/pics/ClassObject.png)

## java.lang.Class类
Class类是对java字节码文件的抽象，每个字节码文件加载到jvm后，jvm都会为其生成一个Class类的实例来描述该字节码文件，且每个类或接口的字节码文件只会生成一个Class类的实例。Class类的定义如下：

    public final class Class<T> implements java.io.Serializable,
                                  GenericDeclaration,
                                  Type,
                                  AnnotatedElement{...}
Class没有公共构造方法。Class对象是在加载类时由Java虚拟机以及通过调用类加载器中的defineClass方法自动构造的。也就是说不需要我们自己去处理创建，JVM已经帮我们创建好了。

## 获取Class对象的三种方式
+ 调用某个对象的getClass()方法会返回该对象所属类的Class对象实例，getClass是Object类的方法。
+ 任何数据类型（即类，包括基本数据类型）都有一个“静态”的class属性。
+ Class类的静态方法：forName(String className)。

## Class类的方法
### 获取构造方法的方法
#### getgetConstructors()、getConstructor(Class<?>... parameterTypes)获取公有访问级别的构造方法
+ getgetConstructors()可以获取到该类的所有公有构造方法构造方法组成的数组：Constructor[]。
+ getConstructor(Class<?>... parameterTypes)可以获取到参数类型为parameterTypes指定的公有构造方法的Constructor实例对象。

#### getDeclaredConstructors()、getDeclaredConstructor(Class<?>... parameterTypes)获取所有访问级别的构造方法
+ getDeclaredConstructors()可以获取到所有的构造方法(包括：私有、受保护、默认、公有)组成的数组：Constructor[]。
+ getDeclaredConstructor(Class<?>... parameterTypes)可以获取到参数类型为parameterTypes指定的所有构造方法(包括：私有、受保护、默认、公有)的Constructor实例对象。

#### newInstance(Object... initargs)调用构造方法
Constructor对象表示的构造方法来创建该构造方法的声明类的新实例，并用指定的初始化参数初始化该实例。


### 获取成员变量并使用
#### getFields()、getField(String name)获取公有访问级别的属性
+ getFields()方法获取类的所有公有访问级别的属性，返回一个属性数组:Field[]。
+ getField(String name)方法获取名字为name的公有属性，返回属性对象：Field。

#### getDeclaredFields()、getDeclaredField(String name)获取所有访问级别的属性
+ getDeclaredFields()方法获取所有的属性(包括：私有、受保护、默认、公有)，返回一个属性数组:Field[]。
+ getDeclaredField(String name)方法获取名字为name的所有的属性(包括：私有、受保护、默认、公有)，返回属性对象：Field。

#### 属性赋值
Field类的set(Object object,Object value)方法，可以为object对象的对应属性设置为value值。
