---
title: maven 常用命令参数
date: 2019-05-15  22:14:26
tags: maven          
category: maven
---
0. `mvn -B archetype:generate  -DgroupId=com.my -DartifactId=simple-weather -DpackageName=com.baicy -Dversion=1.0 `: 创建新项目，可能需要删除 settings配置文件中 archetypeRepository 和 archetypeCatalog 配置。 
2. ` mvn dependency:list` ：输出依赖所有依赖的包
3. ` mvn dependency:tree -Dverbose` ：输出依赖树，解决依赖冲突时有用
4. ` mvn dependency:analyze` ：分析依赖树
5. `-U` :强制更新本地库
6. `mvn clean cobertura:cobertura -Dcobertura.report.format=html`:生成html格式的测试覆盖率报告
7. `mvn help:effective-pom`：生成完整的 POM 文件，包含 super pom
8. `mvn help:effective-settings`: 查看当前 maven 生效的设置
9. 安装本地JAR包到本地仓库：`mvn install:install-file -Dfile= -DgroupId=org.codehaus.mojo -DartifactId=tmaven-versions-plugin -Dversion=2.2 -Dpackaging=jar`
10. `mvn package -DskipTests`: 跳过测试运行
11. `mvn package -Dmvn.test.skip=true`：跳过测试代码编译同时跳过测试运行
12. `mvn test -Dtest=testClass1,testClass2|Random*Test`：运行指定的测试类（|代表或）

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

### 调试自己开发的 Maven 插件
1. 将自己开发的插件 mvn install 到本地
2. 在一个项目X中使用开发的插件，然后，在该项目路径下执行`mvnDebug com.ebay.raptor.build:assembler-maven-plugin:3.0.25-RELEASE:package`，注意插件的版本要一致，不然代码无法匹配，代码断点会失效；执行后会打开一个端口等待 debugger 连接 
3. 在插件开发的IDEA项目中，新建一个 remote 的 debug/run configuration，port修改为上面的port，然后在项目代码中打上断点，然后debug执行即可
