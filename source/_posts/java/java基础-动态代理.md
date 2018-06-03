---
title: java-动态代理
date: 2018-05-10  21:49:35
tags: 动态代理
category: java
---

# Proxy
    public class Proxy implements java.io.Serializable{

        public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h){
                                            ......
                                          }
    }

Proxy类是java反射工具包，主要功能是生成动态代理,通常使用Proxy.newProxyInstance(...)方法创建Proxy对象。

第一个参数是类加载器对象，

第二个参数是一个Class[]数组，表示代理对象可以代理多个接口中的多个方法，代理对象实际上在内部实现了接口数组中的所有方法，可以看成是接口数组中所有接口的子类型，所以代理对象可以被转换成接口数组中的每个接口类型，并调用接口中的方法。

第三个参数是InvocationHandler对象，当代理对象被当成某个被代理接口的实现对象，并调用被代理接口中的方法时，将会调用InvocationHandler中的invoke方法。并把自身对象的引用作为第一个参数传递给它，把被调用的方法对象作为第二个参数，参数对象作为第三个参数。

Proxy的底层工作原理是：
jdk为我们的生成了一个叫$Proxy0（这个名字后面的0是编号，有多个代理类会一次递增）的代理类，这个类文件放在内存中的，我们在创建代理对象时，就是通过反射获得这个类的构造方法，然后创建的代理实例。

# InvocationHandler接口
    public interface InvocationHandler {
        public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable;
    }
InvocationHandler只有一个invoke方法，有三个参数，proxy是通过Proxy对象生成的动态代理对象，method是代理接口的方法对象，args代表参数。
用动态代理的时候一般会实现InvocationHandler接口，在invoke方法中实现通用的代理功能。所有使用了这个InvocationHandler对象的代理对象（上面生成的Proxy对象），在调用代理接口的方法时，都会执行invoke中的代理逻辑。

所以即使没有代理接口的真正实现类，也可以通过代理对象和InvocationHandler实现代理逻辑，此时，没有实现被代理对象的逻辑功能。所以，即使我们只声明一个接口，也可以通过动态代理，为其实现很高端的功能。Mybatis就是如此，只要声明数据库访问接口，通过动态代理功能就能实现数据库操作。基本思想是通过接口名和方法名，在xml文件中找到对应id的sql语句，并执行。


如果有真正的被代理对象，及代理接口的实现类对象，则一般会在InvocationHandler中，保存一个代理接口实现类对象的引用，并在invoke方法中，通过调用Method类的invoke方法，第一个参数设为被代理对象，表示调用被代理对象（真实对象）的实现方法。