---
title: Java基础-Lambda表达式
date: 2019-05-06  17:30:09
tags: Lambda
category: java
typora-root-url: ..\..
---
注：转载自 http://blog.oneapm.com/apm-tech/226.html

### Lambda 表达式简介
我们知道类和对象是 Java 中的一等公民，也就是说Java中几乎所有的语言都是要先定义一个类或接口，然后创建对象，然后调用对象的方法。不像C那样可以直接定义一个函数然后直接调用这个函数而无需创建对象。Java中的方法参数都是对象类型的，如果我想要传入一个函数作为参数，就比较麻烦，通常是使用一个匿名内部类的方式，语法显得很臃肿。Lambda 表达式正是为了解决这一个问题的，它是一种匿名函数(对 Java 而言这并不完全正确，但现在姑且这么认为)，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。

你可以将其想做一种速记，在你需要使用某个方法的地方写上它。当某个方法只使用一次，而且定义很简短，使用这种速记替代之尤其有效，这样，你就不必在类中费力写声明与方法了。

Java 中的 Lambda 表达式通常使用 `(argument) -> (body)` 语法书写，例如：
```
(arg1, arg2...) -> { body }
(type1 arg1, type2 arg2...) -> { body }
```
以下是一些 Lambda 表达式的例子：
```
(int a, int b) -> {  return a + b; } ;
() -> System.out.println("Hello World"); 
(String s) -> { System.out.println(s); } ;
() -> 42 
() -> { return 3.1415 }; 
```
### Lambda 表达式的结构
让我们了解一下 Lambda 表达式的结构。
1. 一个 Lambda 表达式可以有零个或多个参数
2. 参数的类型既可以明确声明，也可以根据上下文来推断。例如：(int a)与(a)效果相同
3. 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：(a, b) 或 (int a, int b) 或 (String a, int b, float c)
4. 空圆括号代表参数集为空。例如：() -> 42
5. 当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a
6. Lambda 表达式的主体可包含零条或多条语句
7. 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
8. 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

### 函数式接口
在 Java 中，Marker（标记）类型的接口是一种没有方法或属性声明的接口，简单地说，marker 接口是空接口。相似地，函数式接口是只包含一个抽象方法声明的接口，可以包含多个默认方法、类方法，但只能有一个抽象方法。

java.lang.Runnable 就是一种函数式接口，在 Runnable 接口中只声明了一个方法 void run()，相似地，ActionListener 接口也是一种函数式接口，我们使用匿名内部类来实例化函数式接口的对象，有了 Lambda 表达式，这一方式可以得到简化。

每个 Lambda 表达式都能隐式地赋值给函数式接口，例如，我们可以通过 Lambda 表达式创建 Runnable 接口的引用。
```
Runnable r = () -> System.out.println("hello world");
```
当不指明函数式接口时，编译器会自动解释这种转化：
```
new Thread(
   () -> System.out.println("hello world")
).start();
```
因此，在上面的代码中，编译器会自动推断：根据线程类的构造函数签名 `public Thread(Runnable r) { }`，将该 Lambda 表达式赋给 Runnable 接口。

以下是一些 Lambda 表达式及其函数式接口：
```
Consumer<Integer>  c = (int x) -> { System.out.println(x) };
BiConsumer<Integer, String> b = (Integer x, String y) -> System.out.println(x + " : " + y);
Predicate<String> p = (String s) -> { s == null };
```
@FunctionalInterface 是 Java 8 新加入的一种接口，用于指明该接口类型声明是根据 Java 语言规范定义的函数式接口。Java 8 还声明了一些 Lambda 表达式可以使用的函数式接口，当你注释的接口不是有效的函数式接口时，可以使用 @FunctionalInterface 解决编译层面的错误。

以下是一种自定义的函数式接口：
```
@FunctionalInterface
public interface WorkerInterface {
   public void doSomeWork();
}
```
根据定义，函数式接口只能有一个抽象方法，如果你尝试添加第二个抽象方法，将抛出编译时错误。例如：
```
@FunctionalInterface
public interface WorkerInterface {
    public void doSomeWork();
    public void doSomeMoreWork();
}
```
错误：
```
Unexpected @FunctionalInterface annotation 
    @FunctionalInterface ^ WorkerInterface is not a functional interface multiple 
    non-overriding abstract methods found in interface WorkerInterface 1 error
```
函数式接口定义好后，我们可以在 API 中使用它，同时利用 Lambda 表达式。例如：
```
 //定义一个函数式接口
@FunctionalInterface
public interface WorkerInterface {
   public void doSomeWork();
}
```
```
public class WorkerInterfaceTest {
    public static void execute(WorkerInterface worker) {
        worker.doSomeWork();
    }

    public static void main(String [] args) {
        //invoke doSomeWork using Annonymous class
        execute(new WorkerInterface() {
            @Override
            public void doSomeWork() {
                System.out.println("Worker invoked using Anonymous class");
            }
        });
        //invoke doSomeWork using Lambda expression 
        execute( () -> System.out.println("Worker invoked using Lambda expression") );
    }
}
```
输出：
```
Worker invoked using Anonymous class 
Worker invoked using Lambda expression
```
这上面的例子里，我们创建了自定义的函数式接口并与 Lambda 表达式一起使用。execute() 方法现在可以将 Lambda 表达式作为参数。
### Lambda 表达式与匿名类的区别
使用匿名类与 Lambda 表达式的一大区别在于关键词的使用。对于匿名类，关键词 this 解读为匿名类，而对于 Lambda 表达式，关键词 this 解读为写就 Lambda 的外部类。

Lambda 表达式与匿名类的另一不同在于两者的编译方法。Java 编译器编译 Lambda 表达式并将他们转化为类里面的私有函数，它使用 Java 7 中新加的 invokedynamic 指令动态绑定该方法，关于 Java 如何将 Lambda 表达式编译为字节码，Tal Weiss 写了一篇很好的文章。