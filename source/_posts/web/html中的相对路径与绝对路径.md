---
title: html中的相对路径与绝对路径
date: 2018-05-11 11:02:00
tags: html
category: web
---
# html中的相对路径与绝对路径
------------------------------
<img src="pics/abslout-relative.gif" alt="相对路径与绝对路径" title="相对路径与绝对路径">

|引用者|      被引用者|     相对路径|                        绝对路径|
|------|      ------|      --------|                        -------|
|Ref1.htm|    BeRef1.gif|  ../SubDir2/BeRef1.gif|        /Dir1/SubDir2/BeRef1.gif|
|Ref2.htm|    BeRef1.gif| ../../Dir1/SubDir2/BeRef1.gif| /Dir1/SubDir2/ BeRef1.gif|
|Ref1.htm|    BeRef2.htm|   ../../Dir2/BeRef2.htm|           /Dir2/BeRef2.htm|
|Ref2.htm|    BeRef2.htm|   ../BeRef2.htm|                   /Dir2/BeRef2.htm|

