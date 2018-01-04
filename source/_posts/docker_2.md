---
title: Docker学习笔记--入门
date: 2017-12-13 23:08:45
tags:
    - Docker
---
!["Docker"](/images/docker.jpg)

### 运行第一个镜像
```bash
sudo docker run ubuntu echo "Hello World"
```
<!--more-->
会输出以下的结果：
```bash
Uable to find image 'ubuntu' locally
ubuntu:latest:The image you are pulling has bean verified
...
...
Status: Dowloaded newer image for ubuntu:latest
Hello World 
```
这究竟发生了什么？</br>在我们调用docker run命令，就启动了一个容器。其中`ubuntu`参数是我们打算使用的镜像。输出的第一行告诉我们本地没有ubuntu镜像。Docker便会在`Dokcer Hub`进行在线搜索，并下载ubuntu最新版本的镜像。镜像下载完成以后，Docker会将它转为容器并运行，然后在容器中执行我们指定的命令`echo "Hello World"`，输出的结果显示在最后一行。

### 请求Docker容器中的Shell
```bash
sudo docker run -t -i  ubuntu bash
root@c92578c38131:/# echo "Hello Docker"
Hello Docker
root@c92578c38131:/# exit
exit
```
这样就会和ssh进入远程主机相似。其中的`-t -i`参数表示我们想要一个交互式的会话，当你退出Shell的时候，容器就会停止- **主进程运行多久，容器就运行多久**

### Docker化

* 首先启动一个容器，并安装一些包：
```bash
sudo docker run -ti --name cowsay --hostname cowsay debian bash
root@cowsay:/# apt-get update
...
Reading package lists... Done
root@cowsay:/# apt-get install -y cowsay fortune
...
root@cowsay:/#
```
* `docker commit` 命令
```bash
sudo docker commit cowsay test/cowsayimage
```
* 验证
```bash
sudo docker run test/cowsayimage /usr/games/cowsay 'Moooo'

 _______
< Moooo >
 -------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
* 使用Dockerfile创建镜像
使用Dockerfile主要是为了`可复用性`，是创建镜像的过程自动化。
```bash
mkdir cowsay
cd cowsay
vim Dockerfile
```
然后将以下内容放入Dockerfile中
>FROM debian:latest</br>
RUN apt-get update && apt-get install -y cowsay fortune

* 生成镜像
```bash
sudo docker build -t test/cowsay-dockerfile .
```

### registry到云仓库中
我使用的是阿里云的仓库服务，简单的说明一下：

* 登录到[阿里云](https://dev.aliyun.com)上的`管理中心`。
* 创建一个命名空间
* 创建一个镜像仓库
* 登录到阿里云docker registry(以我为例)：
```bash
 sudo docker login --username=chenzehao_1949@126.com registry.cn-hangzhou.aliyuncs.com
```
* 将镜像推送到registry(以我为例)：
```bash
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/langzi_1949/docker-images:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/langzi_1949/docker-images:[镜像版本号]
```
* 从registry中拉取镜像(以我为例)：
```bash
$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/langzi_1949/docker-images:[镜像版本号]
```

