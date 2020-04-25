---
title: docker 命令
date: 2019-02-21  20:38:24
tags: docker
category: docker
---
#### docker 核心组件
0. Docker客户端与服务器端：Docker是 C/S 架构的，
1. 镜像：镜像是构建 Docker 世界的基石，Docker基于镜像来运行容器。
2. 容器：容器是基于镜像的，能够为一系列操作提供一个隔离的执行环境。
3. Registry：相当于 GitHub，用来存储、发布镜像的仓库。
4. 如何确定自己是否是登录到一台物理机/虚拟机/容器shell中： systemd-detect-virt -c
   
#### 镜像操作
0. `docker build -t [<仓库名>[:<标签>] 镜像构建上下文路径` : 利用 Dockerfile 来制作镜像
1. `docker search xxx`: 查找镜像
2. `docker pull &lt;image_name&gt;`: 从 Registry 下载镜像
3. `docker push &lt;image_name&gt;`: 上传到 Registry ，默认dockerHub
4. `docker images` : 查看docker镜像
5. `docker image ls -a`: 查看所有镜像
6. `docker image ls &lt;image_name&gt;`: 查看指定镜像
7. `docker image inspect &lt;image_name&gt;`: 查看指定镜像的详细信息
8. `docker image rm [选项] &lt;镜像1&gt; [&lt;镜像2&gt; ...]`: 删除镜像
9. `docker image prune`: 删除虚悬镜像(dangling image)
10. `docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]` : 将修改后的容器保存为镜像

##### Dockerfile 镜像指令
当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。  
Dockerfile 中每一个指令都会建立一层，每一条指令执行过程都类似于：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。
1. FROM: 指定基础镜像 ，后续指令都是基于基础镜像的
2. CMD ：容器就是进程，既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。如果启动容器时，添加了指令，则会覆盖CMD指令
    `CMD ['/bin/bash','-l']`
3. ENTRYPOINT： 类似于CMD，但是启动容器时添加的指令将会作为参数传递到ENTRYPOINT的指令中
4. WORKDIR：在容器内部切换工作目录，CMD和ENTRYPOINT的指令就运行在这个目录中
5. ENV: 用来在构建镜像过程中设置环境变量，这个环境变量可以在后续RUN指令中使用
6. USER：用来指定该镜像会以什么用户运行
7. VOLUME：为基于镜像创建的容器添加卷。
8. ADD: 用来将构建环境下的文件和目录复制到镜像中，不能对构建目录或者上下文之外的文件进行ADD操作。ADD归档文件时会自动解压
    `ADD hello.jar /opt/application/hello.jar`
9. COPY： 类似于ADD，区别在于不会做文件提取和解压的工作。
10. ONBUILD：
11. RUN: 用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。
12. EXPOSE <端口1> [<端口2>...] : 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

#### 容器操作

##### 创建容器
1. docker run &lt;options&gt; &lt;image_name&gt; &lt;command&gt;: 创建并启动容器执行 command 命令，容器是否长久运行和 command 命令有关，相关选项：
    + `-d` :代表后台守护进程运行
    + `-p 宿主机:宿主机端口：容器端口`：将容器端口映射到宿主机端口
    + `--restart=always`：自动重启容器
    + `--name=yourname`：指定容器名字
    + `-v 宿主机目录：容器目录：ro|rw`： 将宿主机的目录作为卷，挂载到容器对应目录
    + `-e, --env=[]`: 设置环境变量  
示例：`docker run -p 3316:3306 -e MYSQL_ROOT_PASSWORD=123456 -d --name=mysql-master  mysql`
2. docker create &lt;options&gt; &lt;image_name&gt; &lt;command&gt;：创建一个新的容器但不启动它。 示例：`docker create  --name myrunoob  nginx:latest`

##### 启动/停止容器
1. docker run：同上
1. docker start &lt;container_id&gt;: 重启已经停止的容器
2. docker stop &lt;container_id&gt;: 停止容器
3. docker restart :重启容器  示例-`docker restart master slave`

##### 容器查看/删除
1. docker ps : 查看正在运行的容器
2. docker container ls: 查看容器（默认指显示正在运行的容器，与ps一样）， -a 选项查看所有容器
3. docker inspect &lt;container_id&gt;: 查看指定容器的详细信息
4. docker system df: 查看镜像、容器、数据卷所占用的空间
5. docker container rm &lt;container_id&gt;: 删除指定容器
6. docker container prune: 清理所有处于终止状态的容器

##### 其它容器操作
1. docker logs &lt;container_id&gt; : 查看容器的运行日志
2. docker top &lt;container_id&gt; : 查看容器内的进程
3. docker exec -it &lt;container_id&gt; &lt;command&gt;: 进入容器运行指定命令
4. 复制宿主机文件到容器目录：`docker cp slave.cnf mysql-slave:/etc/mysql/conf.d`