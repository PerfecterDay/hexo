---
title: maven 总结
date: 2019-08-24  10:33:28
tags: maven          
category: maven
---
Maven 约定项目的主代码位于 src/main/java 目录，测试代码位于 src/test/java 目录；测试代码只会在运行测试时才会用到，打包时不会打包。

### Maven 坐标与依赖配置
Maven定义了一组规则：任何一个构件都可以使用Maven坐标唯一标识，Maven坐标的元素包括：（groupId、artifactId、version，必须定义）、packaging（可选）、classifier（不能直接定义，由插件帮助生成）。

#### 依赖范围
Maven 在编译主代码时会使用一套 classpath1 ，在编译和运行测试代码时，会用到另外一套 classpath2 ，最后，打包（运行）Maven 项目时又是另外一套classpath3 。 实际上 Maven 依赖的 scope 就是用来控制 Maven 依赖与这三种 classpath 的关系的，Scope可以取如下值：
+ compile : 默认值，会把依赖加入到上述三种 classpath 中
+ test : 只在编译、执行测试代码时有效，即上述第二种 classpath2
+ provided : 对于编译和测试时有效，但是运行时无效（不会打包），即会加到 classpath1/classpath2 中
+ runtime : 对于测试和运行 classpath 有效，编译主代码时无效
+ system : 系统依赖范围，与 provided 依赖范围完全一致
<img src="/pics/maven-scope.png" alt="">

#### 依赖范围与传递性依赖
A依赖B，B依赖C，则A依赖C；我们称 A与B时第一直接依赖，B与C是第二直接依赖， A与C是传递性依赖；下表中最左第一列是第一直接依赖范围，表头是第二直接依赖范围，表格中代表结果的传递性依赖范围。
<img src="/pics/maven-dep-propogation.png" alt="">

#### 依赖调解
当依赖冲突时，例如：
A->B->C->X(1.0)/A->D->X(2.0)
依赖调解第一原则：路径最近者优先。上例中 X(1.0) 路径长为3，而X(2.0)长为2，所以X(2.0)会被解析引用。
A-B->Y(1.0)/A->C->Y(2.0)
依赖路径长途相同时，要靠依赖调解第二原则：第一声明优先。B先声明，则用Y(1.0)；C先声明的话就用Y(2.0)。

#### 可选依赖
A->B/B->X(optional)/B->Y(optional): A不会依赖 X和Y。

#### 排除依赖
当我们想去掉某些传递性依赖时，可以使用 exclusions :
```
<dependency>
    <groupId>com.ebayinc.platform.mayfly</groupId>
    <artifactId>mayfly-client</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>uKernelCore</artifactId>
            <groupId>com.ebay.kernel</groupId>
        </exclusion>
        <exclusion>
            <artifactId>netty-common</artifactId>
            <groupId>io.netty</groupId>
        </exclusion>
    </exclusions>
</dependency>
```
exclusion 中只需要指出 groupId 和 artifactId 。

### 仓库
<img src="/pics/maven-repo.png" alt="">
在 settings 文件中，使用 repository 元素配置远程仓库，id为 central 是中央仓库；如果仓库需要认证，使用 server 元素配置认证信息，repository 与 server 之间通过id关联，即 server 配置的是 id 相同的 repository 的认证信息。

#### 从仓库解析依赖的机制
当指定了详细的版本时，会尝试从本地仓库寻找构件，找到则解析成功，否则会遍历所有远程仓库尝试下载改构件。  
如果没有显示指定版本，如使用了RELEASE、LATEST或SNAPSHOT，则会根据更新策略（每日检查更新、永远检查更新、从不。。updatePolicy指定）去读取所有远程仓库的元数据groupid/artifactid/maven-metadata.xml，将其与本地仓库的对应元数据合并后，来决定是使用本地版本或者从远程仓库下载。  
使用快照版本（SNAPSHOT)时，Maven会更具更新策略定时去远程仓库查看是否有时间戳更新的版本，有的话则会下载最新版本使用。

#### 镜像
如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。如果在 seetings 中用 mirror 配置X是Y的镜像，那么所有到Y的请求都会转至该镜像仓库。镜像仓库会完全屏蔽被镜像的仓库，如果镜像仓库停止服务，Maven仍然无法访问被镜像仓库。

### 生命周期和插件
三套相互独立的生命周期： 
+ clean: 清理项目，包含 pre-calen 、 clean 、 post-clean 三个阶段
+ default: 构建项目，阶段很多
+ site: 建立项目站点，pre-site 、 site 、 post-site 、 site-deploy 四个阶段
每个生命周期包含一些阶段，这些阶段是有顺序的，并且后边的阶段会依赖前边的阶段，因此执行后边的阶段，前边依赖的阶段也会执行，用户和Maven最直接的交互方式就是调用这些生命周期的阶段。   
三套生命周期之间是相互独立的，也就是说调用不同生命周期中的阶段不会对其它生命周期有影响，更不会执行其它生命周期内的阶段。

#### 插件目标
<img src="/pics/mvn-goal.png" alt="">
可以在 pom 的 build>plugins>plugin>executions>execution 下为某个插件目标绑定到特定的生命周期阶段上。当多个目标绑定到同一个阶段时，插件的声明顺序会决定目标执行的先后顺序。
mvn [options] <plugin:goal> <phase>
Maven不仅可以执行生命周期的阶段，进而执行绑定在这些阶段上的目标，而且可以直接执行某些插件目标。

### 聚合与继承
当我们的两个Maven项目是相关的，共同作用以构成一个更大的项目，我们可以把他们称为更大项目的模块。如 regs-api/regs-serve/regs-ft。一个简单的需求就是：我们想要一次构建两个模块而不是分别进入到不同模块目录下执行 mvn 命令。Maven 聚合就是为这一需求服务的。    
首先，需要创建另一个模块，并且 packaging 类型必须是 POM ，然后，在 pom 文件中配置 modules，每个 module 的值都是一个相对于当前 pom 的**目录**。

当多个模块之间有许多重复的配置时，可以将他们抽取出来作为一个父模块供这些模块继承共享。父模块的打包类型也必须是 pom，而且不需要源代码之类的目录。子模块继承时，使用 parent 元素指定父模块， parent下的 groupId、artifactId、version 制定了父模块的坐标， relativePath 指定父模块 pom 文件相对与本 pom 的路径，默认值是 ../pom.xml，即上层目录中的 pom.xml 。

父 pom 中一些配置元素是可以被继承的，下边是一个完整的列表：
+ groupId ：项目组 ID ，项目坐标的核心元素；  
+ version ：项目版本，项目坐标的核心元素；  
+ description ：项目的描述信息；  
+ organization ：项目的组织信息；  
+ inceptionYear ：项目的创始年份；  
+ url ：项目的 url 地址  
+ develoers ：项目的开发者信息；  
+ contributors ：项目的贡献者信息；  
+ distributionManagerment ：项目的部署信息；  
+ issueManagement ：缺陷跟踪系统信息；  
+ ciManagement ：项目的持续继承信息；  
+ scm ：项目的版本控制信息；  
+ mailingListserv ：项目的邮件列表信息；  
+ properties ：自定义的 Maven 属性；  
+ dependencies ：项目的依赖配置；  
+ dependencyManagement ：项目的依赖管理配置；  
+ repositories ：项目的仓库配置；  
+ build ：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等；  
+ reporting ：包括项目的报告输出目录配置、报告插件配置等。 

#### 继承下的依赖
当模块A和B都依赖了一些共同的 jar，可以将这些共同的依赖放到父模块的依赖配置下，这样子模块就不用添加这些配置直接继承即可。但是，这样的话，以后新加的子模块C都会依赖这些A和B的依赖，有可能对C是没用的。    
可以使用 dependencyManagement 统一管理这些共同依赖， dependencyManagement 下的依赖不会引入实际的依赖（打包不打），但是可以为一些依赖项配置好版本，这样子元素要使用某一项依赖时，只需要直接添加 groupId、artifactId 的 dependency，版本就是父模块中的版本，当然也可以写上 version ，这样会使用子模块中的版本（类似覆盖）。与 dependencyManagement 还有 pluginManagement 。
