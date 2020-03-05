---
title: maven 常用命令参数
date: 2019-05-15  22:14:26
tags: maven          
category: maven
---

1. ` mvn dependency:tree -Dverbose` ：输出依赖树，解决依赖冲突时有用
2. `-U` :强制更新本地库
3. `mvn clean cobertura:cobertura -Dcobertura.report.format=html`:生成html格式的UT覆盖率报告
4. `mvn help:effective-pom`：生成完整的 POM 文件，包含 super pom
5. `mvn -B archetype:generate  -DgroupId=com.my -DartifactId=simple-weather -DpackageName=com.baicy -Dversion=1.0 `: 创建新项目
6. `mvn help:effective-settings`: 查看当前 maven 生效的设置
7. 安装本地JAR包到本地仓库：`mvn install:install-file -Dfile= -DgroupId=org.codehaus.mojo -DartifactId=tmaven-versions-plugin -Dversion=2.2 -Dpackaging=jar`

### IDEA 添加/删除自定义的 archetype
创建一个项目以后，每次新建项目想以此项目为模板建立新项目，可以将该项目添加到IDEA的 archetype 中。

1. 项目pom中添加:
    ``` 
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-archetype-plugin</artifactId>
        <version>3.0.0</version>
    </plugin>```
2. `mvn archetype:create-from-project` ： 生成archetype
3. 到 target/generated-sources/archetype 目录下执行 `mvn install`
4. 在IDEA新建项目或module的时候，添加 archetype
5. C:\Users\BaIcy\.IdeaIC2019.2\system\Maven\Indices\UserArchetypes.xml 文件中删除添加的自定义的 archetype

