---
title: Kafka 相关问题
date: 2019-05-25 22:02:12
tags: 分布式
category: 分布式
---


+ Kafka Producer如何保证消息可靠
request.required.acks有三个值 0 1 -1
    1. 0:生产者不会等待broker的ack，这个延迟最低但是存储的保证最弱当server挂掉的时候就会丢数据
    2. 1：服务端会等待ack值 leader副本确认接收到消息后发送ack但是如果leader挂掉后他不确保是否复制完成新leader也会导致数据丢失
    3. -1：同样在1的基础上 服务端会等所有的follower的副本受到数据后才会受到leader发出的ack，这样数据不会丢失
