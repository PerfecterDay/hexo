---
title: spring bean 的生命周期
date: 2018-09-12  11:12:34
tags: spring            
category: spring
---

# BeanFactory中bean的生命周期

![BeanFactory 中bean的生命周期](/pics/beanfactory-bean-lifecycle.png)

BeanFactory 实例化一个 `bean` 的步骤：
1. 实例化 `bean` 对象
2. 设置 `bean` 对象的依赖属性（依赖注入）
3. 调用`initializeBean`方法
    1. 调用`invokeAwareMethods`方法： `bean` 如果实现了`Aware`相关接口，则调用`Aware`接口相关的方法（ `BeanNameAware`->`BeanClassLoaderAware`->`BeanFactoryAware`)
    2. 调用`applyBeanPostProcessorsBeforeInitialization`方法：调用注册的所有`BeanPostProcessor`的`postProcessBeforeInitialization`方法处理`bean`
    3. 调用`invokeInitMethods`方法: 
        1. 如果`bean`实现了`InitializingBean`接口，则调用`bean`重写的`afterPropertiesSet`方法
        2. 调用自定义配置的`init-method`方法
    4. 调用`applyBeanPostProcessorsAfterInitialization`方法：调用注册的所有`BeanPostProcessor`的`postProcessAfterInitialization`方法处理`bean`

注意到：自定义的 `bean` 实例化时，会调用 `BeanPostProcessor`相关的方法，`BeanPostProcessor`不一定是 `bean` 本身实现的接口，所有实现`BeanPostProcessor`的类都会在`spring`实例化某个`bean`的时候其作用。就是说，如果我们自定义了一个`BeanPostProcessor`的实现类，则它会在`spring`实例化所有其它`bean`的时候起作用。
    

# ApplicationContext中bean的生命周期

![ApplicationContext 中bean的生命周期](/pics/applicationContext-bean-lifecycle.png)

