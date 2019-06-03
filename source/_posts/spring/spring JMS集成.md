---
title: Spring JMS集成
date: 2019-08-04  10:04:39
tags: spring            
category: spring
---

### 使用JMS API开发的一般套路
1. 首先，从 JNDI 获取 JMS 的受管理对象，即 ConnectionFactory 和稍后要用到的一个或多个 Destination 引用。
2. 使用获取 ConnectionFactory 创建发送消息的 Connection
3. 使用 Connection 创建发送消息的 Session
4. 使用 Session 创建发送消息的 MessageProducer
5. 使用 Session 创建将被发送的消息
6. 使用 MessageProducer 发送的消息到指定的 Destination
7. 最后做资源管理，包括 MessageProducer、Session以及Connection
