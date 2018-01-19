---
title: Docker学习笔记--再入门
date: 2017-12-18 20:09:00
tags:
    - Docker
---
!["Docker"](/images/docker.jpg)

### 主体架构
废话不多说，Docker主要组成部分如下：
!["Docker主体"](/images/docker_4.jpg)
一般来说，周边的技术还有很多：Swarm、Compose、Kubernetes等。
<!--more-->

### 镜像
##### 镜像层
Dockerfile中的每个指令执行后都会产生一个新的`镜像层`，而这个镜像层其实是用来启动容器的。一个新的镜像层的建立，是用上一层的镜像启动容器，然后执行Dockerfile的指令后，把它保存为一个新的镜像。
如果在一个Dockerfile中写下：
```bash
FROM debian:latest

RUN apt-get update && apt-get install -y cowsay fortune
```
然后执行，会发现：
```bash
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM debian:latest
 ---> da653cee0545
Step 2/2 : RUN apt-get update && apt-get install -y cowsay fortune
 ---> Running in cf4be5720ad4
......
Removing intermediate container cf4be5720ad4
 ---> 0615075d910e
Successfully built 0615075d910e
Successfully tagged test/cowsay-dockerfile1:latest
```
观察上面，可以发现`cf4be5720ad4` 这个镜像层，启动了一个临时容器，最后又被remove了。
PS：当执行`docker build`的时候，Docker会查看FROM指令所指向的镜像，如果本地没有镜像，Docker会试图下载他；本地已有该镜像，Docker就会使用他，不会再检查是否是最新版本的。

### 容器相连
假设你正在一个容器中运行 Web 服务器。你如何使外界能访问它呢？答案是通过 -p 或 -P选项来“发布”端口。这个命令能够将主机的端口转发到容器上：
```bash
sudo docker run -d -p 8000:80 nginx
...
curl localhost:8000
...
```
其中的 `-p 8000:80` 参数告诉 Docker 将主机的 8000 端口转发至容器的 80 端口。

### 数据卷
利用数据卷和数据容器管理数据，他其实就是一个目录，但不属于UFS的一部分。

##### 方法1:docker -v

* 首先在执行docker时，通过`-v`选项来宣告一个数据卷：
```bash
sudo docker run -it --name container-test -h CONTAINER -v /data debian /bin/bash
root@CONTAINER:/# ls /data/
root@CONTAINER:
```

* 在新的shell中执行`docker inspect`命令，找出数据卷在主机上的实际位置。
```bash
sudo docker inspect -f {{.Mounts}} container-test
[{volume 1f7..33 /mnt/sda1/var/lib/docker/volumes/1f7..33/_data /data local  true }]
```
然后我们就会发现容器的 /data/ 卷仅仅是一个指向主机中 /var/lib/docker/volumes/5cad…/_data 目录的连接

##### 方法2：Dockerfile中使用VOLUME指令
```bash
FROM debian:latest
VOLUME /data
```
这个方法和第一种的效果是相同的。
如果要在Dockerfile设置数据卷的权限，以下设置不会产生效果:
```bash
FROM debian:latest
RUN useradd foo
VOLUME /data
RUN touch /data/x
RUN chown -R foo:foo /data
```
以上是不会有效果的，因为这些效果都是在临时容器中执行产生的，当命令结束的时候，这个卷也会被删除。
正确的方式应该是以下：
```bash
FROM debian:latest
RUN useradd foo
RUN mkdir /data && touch /data/x
RUN chown -R foo:foo /data
VOLUME /data
```

##### 方法3：docker -v 拓展
通过这个方法直接指定主机的目录，命令格式为`-v HOST_DIR:CONTAINER_DIR`。
```bash
sudo docker run -v /home/adrian/data:/data debian ls /data
```

### 数据容器
这种容器的目的就是为了与其他容器进行数据共享。
```bash
sudo docker run --name dbdata postgres echo 'Data-only container for postgres'
```
这个命令就是从postgres中创建一个容器，并且初始化镜像中定义的所有数据卷，执行执行`echo`退出（没有必要在后台一直运行，浪费资源）。
现在就可以通过`--volume-from`参数，使得其他容器也能够使用这个数据卷。
```bash
sudo docker run -d  --volume-form dbdata --name db1 postgres
```


