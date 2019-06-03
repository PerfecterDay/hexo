---
title: springboot加载多环境配置文件
date: 2018-10-10  11:24:48
tags: springboot            
category: springboot
---

### profile配置
在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：
1. application-dev.properties：开发环境 
2. application-test.properties：测试环境 
3. application-prod.properties：生产环境

至于哪个具体的配置文件会被加载，需要在application.properties文件中通过spring.profiles.active属性来设置，其值对应{profile}值。 

### 常用配置
1. `server.port=9090` : 配置端口
2. `spring.mvc.servlet.load-on-startup=1`： 配置启动初始化 Servlet
3. ```logging.path=/user/local/log
    logging.level.com.favorites=DEBUG
    logging.level.org.springframework.web=INFO
    logging.level.org.hibernate=ERROR```：日志设置
4. ```spring.datasource.url=jdbc:mysql://localhost:3306/test
    spring.datasource.username=root
    spring.datasource.password=root
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver```:数据库配置
5. `spring.mvc.static-path-pattern=/resources/**`: 配置静态文件的URL映射
6. `spring.resources.static-locations=classpath:/mystatic`: 配置静态文件的目录
