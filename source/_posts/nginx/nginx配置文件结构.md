---
title: nginx配置文件结构
date: 2018-05-11 11:02:00
tags: nginx
category: nginx
---

# nginx配置文件结构
------------------------
nginx由配置文件中的指令控制，通过指令可以配置nginx的行为。nginx指令分为简单指令和块指令。

1. 简单指令：由配置项名字+由空格分隔的若干参数+分号；组成。
2. 块指令：与简单指令的结构类似，但是由包含指令的大括号{指令}结束，即配置项名字+由空格分隔的若干参数+{指令}（参数可以为空）。如果一个块指令的大括号内能由其他指令，则这个块称为一个上下文context。例如events、http、server、location。
3. 配置文件中所有不在某个上下文中的指令，默认为在main上下文中。
4. events和http指令在main上下文中，server在http中，location在server中。
5. #后边的内容表示注释。