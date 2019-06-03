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

#### 镜像操作
0. docker build: 利用 Dockerfile 来制作镜像
1. docker search xxx: 查找镜像
2. docker pull &lt;image_name&gt;: 从 Registry 下载镜像
3. docker push &lt;image_name&gt;: 上传到 Registry ，默认dockerHub
4. docker images : 查看docker镜像
5. docker image ls: 查看所有镜像
6. docker image ls &lt;image_name&gt;: 查看指定镜像
7. docker image inspect &lt;image_name&gt;: 查看指定镜像的详细信息
8. docker image rm [选项] &lt;镜像1&gt; [&lt;镜像2&gt; ...]: 删除镜像
9. docker image prune: 删除虚悬镜像(dangling image)

#### Dockerfile 镜像指令
0. CMD ：CMD用于指定一个容器启动时需要运行的命令。如果启动容器时，添加了指令，则会覆盖CMD指令
    CMD ['/bin/bash','-l']
1. ENTRYPOINT： 类似于CMD
2. WORKDIR：在容器内部切换工作目录，CMD和ENTRYPOINT的指令就运行在这个目录中
3. ENV: 用来在构建镜像过程中设置环境变量，这个环境变量可以在后续RUN指令中使用
4. USER：用来指定该镜像会以什么用户运行
5. VOLUME：为基于镜像创建的容器添加卷。
6. ADD: 用来将构建环境下的文件和目录复制到镜像中，不能对构建目录或者上下文之外的文件进行ADD操作。ADD归档文件时会自动解压
    ADD hello.jar /opt/application/hello.jar
7. COPY： 类似于ADD，区别在于不会做文件提取和解压的工作。
8. ONBUILD：

#### 容器操作
1. docker run &lt;image_name&gt; &lt;command&gt;: 启动容器执行 command 命令，容器是否长久运行和 command 命令有关，创建容器
    + `-d` :代表后台守护进程运行
    + `-p 宿主机:宿主机端口：容器端口`：将容器端口映射到宿主机端口
    + `--restart=always`：自动重启容器
    + `--name=yourname`：指定容器名字
    + `-v`： 将宿主机的目录作为卷，挂载到容器里，格式为 宿主机目录：容器目录：ro|rw
2. docker ps : 查看正在运行的容器
3. docker inspect &lt;container_id&gt;: 查看指定容器的详细信息
4. docker stop &lt;container_id&gt;: 停止容器
5. docker start &lt;container_id&gt;: 重启已经停止的容器
6. docker container rm &lt;container_id&gt;: 删除指定容器
7.  docker container prune: 清理所有处于终止状态的容器
8. docker system df: 查看镜像、容器、数据卷所占用的空间
9. docker exec -it &lt;container_id&gt; &lt;command&gt;: 进入容器运行指定命令
10. docker logs &lt;container_id&gt; : 查看容器的运行日志
11. docker top &lt;container_id&gt; : 查看容器内的进程 

docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3307:3306 mysql