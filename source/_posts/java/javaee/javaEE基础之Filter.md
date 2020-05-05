---
title: JavaEE基础之 Filter
date: 2019-04-28  17:30:12
tags: web
category: JavaEE
---
## 创建 FIlter
Filter 主要用来在 Servlet 处理请求的前后完成某些操作。

创建 Filter 只要实现 `Filter` 接口即可。过滤器在初始化时将调用 `init()` 方法，可以访问过滤器配置、初始化参数和 ServletContext ,和 Servlet 的 `init()` 方法一样。类似的，应用程序关闭时也会调用 `destory()` 方法。

当请求进入到 Filter 时， 过滤器的 `diFilter()` 方法将会被调用，在该方法中提供了对 ServletRequest 、 ServletResponse 和 FilterChain 对象的访问。因此可以对请求和相应进行处理。

![过滤器链](/pics/filterchain.png)
过滤器炼的工作方式非常像栈。当请求进入时，首先进入第一个过滤器，该过滤器被添加到栈中。当过滤器调用 `FilterChain.diFilter()` 时，下一个过滤器将被添加到栈中，一直到请求进入Servlet中，它是最后一个被加入到栈的元素。

当Servlet的 `service()` 方法返回时，Servlet出栈，然后控制权返回最后一个加入到栈的 Fliter 中，当它的 `diFilter()` 方法返回时，过滤器将从栈中移除，控制权返回到之前的过滤器中，一直到第一个过滤器。当第一个过滤器的 `diFilter()` 方法返回时，请求处理就完成了。
## 配置 Filter
可以是用3种方式配置：
1. 使用部署描述符

在部署描述符中添加
```
<filter>
    <filter-name>myFilter</filter-name>
    <listener-class>
    com.journaldev.filter.MyFilter
    </listener-class>
    <url-pattern>/foo</url-pattern>
    <url-pattern>/bar</url-pattern>
    <servlet-name>helloServlet</servlet-name>
    <dispatcher>REQUEST</dispatcher>
    <init-param>
      <param-name>count</param-name>
      <param-value>5</param-value>
    </init-param>
</filter>
```
与Servlet不同的是，过滤器不是在第一个请求到达时加载的，过滤器的 `init()` 方法总是在应用程序启动时被调。在 `ServletContextListener` 初始化之后，Servlet初始化之前，它们将按部署描述符中的顺序加载。处理请求时也按顺序处理。

2. 使用 `@WebFilter` 注解

在实现了 Filter 接口的类上标注 `@WebFilter` 注解，即可将这个类声明为 Filter .
```
@WebFilter(filterName = "FilterDemo02", 
urlPatterns = { "/*" }, 
initParams = { @WebInitParam(name = "name", value = "xc"),
        @WebInitParam(name = "like", value = "java") })
public class FilterDemo02 implements Filter {
    ....
}
```
注解配置的缺点在于：无法指定各个Filter的加载顺序，也就无法指定各个Filter处理请求的顺序。而这同通常是很重要的。

3. 编程式配置

`ServletContext` 接口提供了动态添加 Filter 的方法：
```
1. javax.servlet.FilterRegistration.Dynamic addFilter(String var1, String var2); 
2. javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Filter var2); 
3. javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Class<? extends Filter> var2);
```
与编程式添加 Servlet 和 Listener 一样，这必须要在在 `ServletContext` 配置完成之前完成，因为容器会根据 `ServletContext` 配置决定加载哪些 Listener/Filter/Servlet. 所以要在 `ServletContextListener` 的 `contextInitialized()` 方法或者 `ServletContainerInitializer` 中的 `onStartup()` 中注册 Listener。

## 过滤器排序