---
title: java基础-网络编程基础
date: 2019-08-25  10:15:33
tags: java          
category: java
---

### Java的基本网络支持

#### InetAddress
Java 使用 InetAddress 来代表 IP 地址，InetAddress 有两个子类：Inet4Address 和 Inet6Address 。分别代表 IPv4 和 IPv6 地址。

InetAddress 没有提供构造器，而是提供了如下两个静态方法来获取 InetAddress 对象：
+ getByName(String name) ：根据主机域名获取对应 InetAddress 对象；
+ getByAdress(byte[] addr): 根据原始 IP 地址获取InetAddress对象；

#### URLEncoder 和 URLDecoder
URLEncoder 和 URLDecoder 提供了静态的工具方法 encode 和 decode 来进行 URL 的编码和解码。

#### URL、URLConnection 和 URLPermission

#### 使用ServerSocker创建TCP服务器端
Java使用 ServerSocket 来创建服务端用于监听客户端连接的套接字。有如下构造方法：
+ ServerSocket(oint port)

#### 使用Socket进行通信
