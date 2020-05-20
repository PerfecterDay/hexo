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
<xml version="1.0" encoding="utf-8"></xml>
<archetype-descriptor name="springboot-archetype-quickstart">
    <fileSets>
        <fileSet filtered="true" packaged="true">
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
        <fileSet filtered="true" packaged="true">
            <directory>src/test/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
        <fileSet filtered="true" packaged="false">
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
            </includes>
        </fileSet>
    </fileSets>
    <requiredProperties>
        <requiredProperty key="port"></requiredProperty>
        <requiredProperty key="groupId">
            <defaultValue>com.baicy.wang</defaultValue>
        </requiredProperty>
    </requiredProperties>
</archetype-descriptor>
```
上述描述符中定义了名称为 quickstart 的 Archetype ，主要包含 fileSets 和 requiredProperties 两个元素。
1. `<fileSets>`
    包含一个或多个 fileset，每个 fileset 定义一个目录及与该目录相关的包含或排除规则。上述代码段中第一个 fileSet 指向的是 src/main/java ，该目录对应的是该Archetype项目的 archetype-resources/src/main/java 子资源目录， filtered 表示是否对该文件集合应用属性替换（${x}这样的值是否替换为命令行输入的 x 参数的值）； packaged 表示是否将该目录下的内容放到生成项目的包路径下。
    include 子元素声明哪些内容将会被包含，** 表示匹配任意目录，* 表示匹配处路径分隔符之外的任意字符。
    <img src="/pics/archetype-resource.png" alt="">

2. `<requiredProperties>` 
   一般在生成 maven 项目时默认需要四个参数：groupId、artifactId、version和package，requiredProperties 用来配置要求额外参数，比如上面要求port参数

项目编写完成后，使用 `mvn clean install` 安装到本地库，就可以通过该 Archtype 的项目坐标来生成项目了：  
`mvn archetype:generate  -DarchetypeGroupId=com.baicy.wang -DarchetypeArtifactId=springboot-archetype-quickstart -DarchetypeVersion=1.0-SNAPSHOT`。

#### Archetype Catalog
当用户以不指定 Archetype 坐标的方式使用 maven-archetype-plugin 的时候，会得到一个 Archetype 的列表供选择，这个列表来源于一个 archetype-catlog.xml 的文件。 maven-archetype-plugin 可以从下面几个地方获取这个文件：
+ internal : maven-archetype-plugin 内置的 Archetype Catalog ，大约包含了 58 个 archetype
+ local : 从用户本地的 ～/.m2/archetype-catlog.xml 加载，该文件默认不存在。
+ remote : 指向了 Maven 中央仓库的 Archetype Catalog。
+ file://... : 用户可以指定本机任意路径下的 archetype-catlog.xml 文件
+ http://... : 用户可以使用HTTP协议指定远程的 archetype-catlog.xml 文件

执行`mvn archetype:generate` 时，使用 `archetypeCatalog` 参数指定插件使用的 catalog ，多个 catalog 源之间使用逗号分隔。该参数默认值是“remote,local“，所以 maven-archetype-plugin 默认使用中央仓库加本机的 catalog 信息。

#### 生成本地仓库的 Archetype Catalog
当我们构建了一个 Archetype 项目并且安装到本地后，我们可以将其添加到本地 archetype-catlog.xml 文件中，这样在生成项目时，就会出现在供选择的列表中了。还有一种方法就是执行 `mvn archetype:crawl` 目标，它会遍历本地的 Maven 仓库的内容，并自动生成 archetype-catlog.xml 文件。