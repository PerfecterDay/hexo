---
title: springboot加载多环境配置文件
date: 2018-10-10  11:24:48
tags: springboot            
category: springboot
---


在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：
1. application-dev.properties：开发环境 
2. application-test.properties：测试环境 
3. application-prod.properties：生产环境

至于哪个具体的配置文件会被加载，需要在application.properties文件中通过spring.profiles.active属性来设置，其值对应{profile}值。 
