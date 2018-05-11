---
title: npm教程
date: 2018-05-11 11:02:00
tags: npm
category: web
---

# npm教程
---------

## npm安装
npm是随同NodeJS一起安装的包管理工具，类似于java开发中Maven。

## npm配置查看及更改
1. *npm config list*：查看npm配置项。
2. *npm config ls -l*：查看npm所有配置项。
3. *npm set key = value*：更改配置项key的值为value，*npm set registry="https：//registry.npm.taobao.org/"*->修改npm中央仓库。

## npm安装依赖包
npm作为一款js的包管理工具,主要作用是对前端项目的依赖包进行管理。

1. *npm init*命令可以创建一个npm包，类似于java中新建一个maven项目。根据提示，会生成一个package.json文件描述了包的信息，包括名称、版本、作者等信息，类似于maven中的pom文件。
2. *npm install name -g*：全局安装name包，*npm install name --save*：安装包的同时，将信息写入package.json依赖中，*npm install name --dev-save*：将报信息写入devDependencies依赖项内。
3. *npm remove name*：移除对name包的依赖，*npm update name*：更新name依赖包。
4. package.json的scripts中添加脚本名字和脚本命令后，使用*npm run 脚本名称* 运行相应脚本。*start*脚本可以省略run：*npm start*。
5. *npm root*：查看当前包安装的路径，*npm root -g*：查看全局包的安装路径。
