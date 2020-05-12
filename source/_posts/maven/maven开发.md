---
title: maven plugin 开发和 archetype 
date: 2020-05-14  20:45:23
tags: maven          
category: maven
---
### Maven 插件开发
#### Maven 插件开发的一般步骤
1. 创建一个 maven-plugin项目：使用 `maven archetype:generate` ，然后选择 maven-archetype-plugin 快速创建一个插件项目
2. 为插件编写目标：每个插件必须包含一个或多个目标，Maven称之为 Mojo，继承自 AbstractMojo 类。
3. 为目标提供配置点：在编写Mojo时，提供可配置的参数
4. 编写代码实现目标行为：根据实际需求实现Mojo
5. 错误处理及日志
6. 测试插件：编写自动化的测试代码测试插件。

Maven插件的 pom 有两个特殊的地方：
  1. packaging 类型必须是 maven-plugin
  2. 必须依赖一个 maven-plugin-api 的 artifact。
创建好插件项目之后，我们要创建一个Mojo：继承 AbstractMojo 、实现 execute() 方法、提供 @goal 注解。


### Maven Archetype 开发
一个典型的 Maven Archetype 项目主要包括以下几个部分：
+ pom.xml: Archetype 项目自身的 pom
+ src/main/resources/archetype-resources/pom.xml : 基于该Archetype生成的项目的 pom 文件
+ src/main/resources/META-INF/maven/archetype-metadata.xml : Archetype 的描述符文件
+ src/main/resources/archetype-resources/** : 其它需要包含在 Archetype 中的内容。
一个 Archetype 最核心的是 archetype-metadata.xml 描述符文件，它主要用来控制两个事情：
1. 声明哪些文件和目录应该包含在 Archetype 中
2. 这个 Archetype 使用哪些属性参数
一个示例描述符文件文件：
```
<archetype-descriptor
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0
        https://maven.apache.org/xsd/archetype-descriptor-1.1.0.xsd"
        xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        name="quickstart">
    <fileSets>
        <fileSet filtered="true" packaged="true">
            <directory>src/main/java</directory>
        </fileSet>
        <fileSet>
            <directory>src/test/java</directory>
        </fileSet>
    </fileSets>
</archetype-descriptor>
```
+ <requiredProperties> : 
+ <fileSets> : 包含一个或多个 fileset，每个 fileset 定义一个目录及与该目录相关的包含或排除规则
+ <modules> : 模块定义