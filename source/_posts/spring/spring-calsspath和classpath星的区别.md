---
title: classpath和classpath*的区别
date: 2018-06-01 13:55:12
tags: spring
category: spring
---

# classpath和classpath*的区别

记录一下踩坑：

有两个项目A和B，A依赖B，最终运行的是A。使用mybatis做数据库访问，在一个B的jar包中写好了mapper.xml文件，A项目运行时，能运行A中的mapper文件，找不到B中绑定的sql语句。就是说mybatis扫描mapper.xml文件时，能扫描到项目本身的mapper，却不能扫描到依赖jar包中的mapper，仔细查看文件才发现配置文件有问题：

    mapper-locations: classpath:mapper/**/*.xml
改为

    mapper-locations: classpath*:mapper/**/*.xml
后，问题解决。

classpath和classpath*区别： 
1. classpath：只会到你的class路径中查找找文件。

2. classpath*：不仅包含class路径，还包括jar文件中（class路径）进行查找。

注意： 用classpath*:需要遍历所有的classpath，所以加载速度是很慢的；因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用classpath*。