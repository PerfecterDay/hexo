---
title: docker 命令
date: 2019-02-21  20:38:24
tags: docker
category: docker
---

0. docker build: 利用 Dockerfile 来制作镜像
1. docker images : 查看docker镜像
2. docker pull <image_name>: 从 dockerHub 下载镜像
3. docker image rm [选项] <镜像1> [<镜像2> ...]: 删除镜像
4. docker image prune: 删除虚悬镜像(dangling image)
5. docker ps : 查看正在运行的容器
6. docker run <image_name>: 启动容器
7. docker stop <container_id>: 停止容器
8. docker container rm <container_id>: 删除指定容器
9. docker container prune: 清理所有处于终止状态的容器
10. docker system df: 查看镜像、容器、数据卷所占用的空间