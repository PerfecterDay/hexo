---
title: shiro中的`FilterChainManager`分析
date: 2018-06-04 22:33:40
tags: shiro
category: shiro
---


# shiro中的`FilterChainManager`分析

## `FilterChainManager`接口
`FilterChainManager`主要提供了管理`FilterChain`的功能。定义如下：

    public interface FilterChainManager {
        Map<String, Filter> getFilters();
        NamedFilterList getChain(String var1);
        boolean hasChains();
        Set<String> getChainNames();
        FilterChain proxy(FilterChain var1, String var2);
        void addFilter(String var1, Filter var2);
        void addFilter(String var1, Filter var2, boolean var3);
        void createChain(String var1, String var2);
        void addToChain(String var1, String var2);
        void addToChain(String var1, String var2, String var3) throws ConfigurationException;
    }
`FilterChain`是 `javax.servlet`包下的标准对象类。

## `DefaultFilterChainManager`
shiro中只有一个默认的`FilterChainManager`的实现，即`DefaultFilterChainManager`：

    public class DefaultFilterChainManager implements FilterChainManager {
        private FilterConfig filterConfig;
        private Map<String, Filter> filters = new LinkedHashMap();
        private Map<String, NamedFilterList> filterChains = new LinkedHashMap();
        .....
    }
其中，`NamedFilterList`类定义如下：

    public interface NamedFilterList extends List<Filter> {
        String getName();
        FilterChain proxy(FilterChain var1);
    }
可以看出，该类对标准的`FilterChain`进行了封装，将一个`FilterChain`对象和一个名字关联起来，名字通过`getName()`方法获取。

`DefaultFilterChainManager`

