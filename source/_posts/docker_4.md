---
title: Docker学习笔记--命令
date: 2017-12-22 00:01:00
tags:
    - Docker
---
!["Docker"](/images/docker.jpg)

#### run命令
启动容器必须使用的命令。

* -d 使得容器在后台运行，命令返回的是容器的ID
* -i 保持`stdin`打开，一般与`-t`一起使用，用作启动交互式会话的容器
* --restart 重启容器，可以配置失败次数，on-failure:10 意味着当退出值不为0时，最多尝试10次重启
* --rm 退出时自动删除容器。不能与 -d 选项同时使用
<!--more-->
* -t 分配一个伪终端，与`-i`一起使用
* -e 设置容器中的环境变量
* --name 设置容器的名称，方便称呼
* -v 设置数据卷
* --link 建立一个与指定容器连接的内部网络接口
* -p “发布”容器的端口，使主机能访问它

#### 容器管理命令
在容器生命周期中，以下命令也可以进行管理容器。

* docker attach [OPTIONS] CONTAINER
* docker create 从镜像中创建一个容器，但不会启动它
* docker cp 在容器和主机之间复制文件和目录
* docker exec 在容器中执行一个命令
```bash
sudo docker exec bb125f977f51 echo 'Hello'
sudo docker exec -ti  bb125f977f51 /bin/bash
root@bb125f977f51:/# ls
```
* docker rm 删除容器
* docker start 启动容器
* docker stop 停止容器，但不是删除

#### Docker信息命令
主要就是`docker help`,`docker info`,`docker version`等

#### 容器信息命令
以下命令提供更多有关运行中及已停止的容器信息。
* docker diff
```bash
##对比容器所使用的镜像，显示容器的文件系统的变化。例如：
ID=$(docker run -d debian touch /NEW-FILE)
docker diff $ID
A /NEW-FILE
```

* docker inspect 把容器或镜像作为参数，获取它们的详细信息。这些信息包括大部分配置信息、联网设
置以及数据卷的映射信息 
* docker logs 输出容器的日志
* docker ps 输出当前运行的容器的信息
* docker top 把容器作为参数，提供该容器内运行中进程的信息

#### 镜像管理命令
下面的命令用于镜像的创建和处理。
* docker build 从Dockerfile中创建镜像
* docker commit 从指定的容器中创建镜像
* docker history 输出镜像中每个镜像层的信息
* docker images 列出本地所有的镜像
* docker rmi 删除指定的一个或多个镜像

#### 使用寄存服务器命令
* docker login 在指定的寄存服务器进行注册或登录
* docker logout 从 Docker 寄存服务器注销
* docker pull 从寄存服务器下载指定的镜像
* docker push 将镜像或仓库推送到寄存服务器
* docker search 列出 Docker Hub 上匹配搜索词的公共仓库（通常直接在网上查询）
