---
title: ActiveMQ 入门
date: 2019-08-04  10:26:39
tags: mq            
category: mq
---

### 安装
直接去官网下载压缩包，解压即可.

### 启动
`\bin\activemq start`
使用下面命令验证启动是否成功：
`netstat -an|find '61616'`

### 监控
`http://localhost:8161/admin`
默认用户名和密码： admin/admin ，可以在 `conf/jetty-real.properties` 文件中配置

### 停止
`bin/activemq stop` 
或者
```
ps -ef|grep activemq
kill [PID]
```

1570016807