---
title: powermock的使用
date: 2018-08-24 15:11:23
tags: test
category: test
---


## mock
Powermockito.mock()方法主要是根据 class 参数创建一个对应的 mock 对象， powermock 的创建方式可不像 easymock 等使用 proxy 的方式创建，他是会在你运行的过程中动态的修改 class 字节码文件的形式来创建的。

## do..when..then
我们可以看到，每次当我们想给一个 mock 的对象进行某种行为的预期时，都会使用 do...when...then...这样的语法，其实理解起来非常简单:做什么、在什么时候、然后返回什么。

## verify
当我们测试一个 void 方法的时候，根本没有办法去验证一个 mock 对象所执行后的结果，因此唯一的方法就是检查方法是否被调用，在后文中将还会专门来讲解。

## mock局部变量
    public class EmployeeService {
        public int getTotalEmployee() {
            EmployeeDao employeeDao = new EmployeeDao(); 
            return employeeDao.getTotal();
        }
    }
假设要测试上述方法，但是`EmployeeDao`对象并不可用且不是注入到`EmployeeService`的成员变量中的，我们无法mock一个dao然后设置到`EmployeeService`的成员变量上。

    EmployeeDao employeeDao = PowerMockito.mock(EmployeeDao.class); try {
    PowerMockito.whenNew(EmployeeDao.class).withNoArguments() .thenReturn(employeeDao);
    PowerMockito.when(employeeDao.getTotal()).thenReturn(10);

    Mockito.verify(employeeDao).addEmployee(employee);
其实相当于mock了`EmployeeDao`的构造函数。最后一行的`verify`方法可以验证`addEmployee`方法是否被调用了。

## 
