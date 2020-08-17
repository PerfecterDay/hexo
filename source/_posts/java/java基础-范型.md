---
title: java基础-范型
date: 2019-03-04  22:08:24
tags: 范型
category: java
typora-root-url: ..\..
---
## 范型类
Java 集合有个缺点：当我们把一个对象放入集合中时，集合就会忘记这个对象的数据类型，当再次取出对象时，对象的编译类型就变成了Object类型，必须进行强制类型转换才能转换成需要的编译类型。 

所谓范型：就是允许在定义类、接口时指定类型参数(在类名后边加上一对尖括号，类型形参放在尖括号内，多个形参用逗号分隔)，这些类型形参将在声明变量、创建对象时确定（即传入实际的类型参数），类型形参在在整个类内部可用，几乎所有可以使用普通类型的地方都可以使用这种类型形参。JDK1.5之后改写了所有集合类支持范型。

使用范型类时，可以传入不同的类型实参创建对象，如 List<Object>、List<String>、List<Integer>...，效果上，相当于创建了集合类的一个子类，子类只能存放特定的类型，但是系统并没有为这些声明生成新的 class 文件，更不会把他们当成新类来处理，它们是同样的 List 类。List 类的静态变量和方法会在所有实例之间共享，所以在静态方法、静态初始化块或者静态成员变量的声明和初始化块中不允许使用类型形参。自然，这些效果上的子类之间更不会存在继承关系，也就是说 List<String>、List<Integer> 不是 List<Object> 的子类。所以下述代码会出现编译错误：
```
public void test(List<Object> list){...}//严格要求传入 List<Object> 范型
List<String> sList = new ArrayList();
test(sList);
```
当创建了带范型声明的接口、父类之后，可以创建接口的实现类，或者从父类来派生子类，如果实现类或者派生子类不是范型类，那么当使用这些接口、父类时，不能再包含类型形参，必须指定具体的类型实参；如果实现类或者派生子类是范型类，则可以为父类指定相同的类型参数。如下面的代码是错误的：
```
class Creatrue<T>{}
class Man extends Creatrue<T>{} //编译错误
class Man extends Creatrue<String>{} //可以正确编译
class Man<T> extends Creatrue<T>{} //可以正确编译
class Man<T> extends Creatrue<String>{} //可以正确编译
```
## 范型通配符
将 ? 作为类型实参传递范型类，表示匹配任何类型实参类型。上面的代码改成下面这样可以编译通过：
```
public void test(List<?> list){...}//可以
List<String> sList = new ArrayList();
test(sList);
```
`List<? extends Shape>` 表示匹配所有Shape及其子类类型实参的范型。
`List<? super Shape>` 表示匹配所有Shape及其父类类型实参的范型。

## 范型方法
即使在定义类、接口时没有使用范型，但是在定义方法时想定义类型形参，这也是可以的。
```
修饰符 <T,S> 返回值类型 方法名（形参列表）{
    //方法体
}
```
类型参数 T,S只能在该方法中使用。与类型范型不同的是，在调用范型方法时，无需显示的传入类型实参，系统可以直接推断出类型形参的类型。
```
public <T> void arrayToCollection(T[] arr, Collection c){
    for(T v:arr){
        c.add(v);
    }
}
Integer[] arr3 = new Integer[10];
List<Integer> list2 = new ArrayList<>();
arrayToCollection(arr3,list2);
```
范型方法允许类型参数被用来表示方法的多个参数之间的类型依赖关系，或者方法返回值和参数之间的类型依赖关系。如果没有这样的依赖关系，不应该使用范型方法。