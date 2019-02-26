---
title: docker 命令
date: 2019-02-21  20:38:24
tags: docker
category: docker
---
#### 镜像操作
0. docker build: 利用 Dockerfile 来制作镜像
1. docker pull &lt;image_name&gt;: 从 dockerHub 下载镜像
2. docker images : 查看docker镜像
3. docker image ls: 查看所有镜像
4. docker image ls &lt;image_name&gt;: 查看指定镜像
5. docker image rm [选项] &lt;镜像1&gt; [&lt;镜像2&gt; ...]: 删除镜像
6. docker image prune: 删除虚悬镜像(dangling image)

#### 容器操作
5. docker ps : 查看正在运行的容器
6. docker run &lt;image_name&gt; &lt;command&gt;: 启动容器执行 command 命令，容器是否长久运行和 command 命令有关
7. docker stop &lt;container_id&gt;: 停止容器
8. docker container rm &lt;container_id&gt;: 删除指定容器
9.  docker container prune: 清理所有处于终止状态的容器
10. docker system df: 查看镜像、容器、数据卷所占用的空间
11. docker exec -it &lt;container_id&gt; &lt;command&gt;: 进入容器运行指定命令