---
title: Docker学习笔记--安装
date: 2017-12-10 08:09:23
tags:
    - Docker
---

!["Docker"](/images/docker.jpg)

### 背景
容器化技术正在如火如荼的开展着，其中Docker就是其中的集大成者，所以学习Docker迫在眉睫。

----
### 容器和虚拟机
话不多说，直接上图！
!["Container Vs VM"](/images/docker_1.jpg)

---
<!--more-->
### 安装

* #### Linux 和 Mac
由于linux和mac有点天然的优势，他们安装docker是异常的方便，随便在网上找一个教程就行了，这边就不赘述了。

* #### Windows
* ##### Windows 10+
win10也不用说了，docker官方是支持win10的，直接下载`docker for windows`就行了。

* ##### Windows7
win7目前使用率还是挺高的，那我们怎么学习Docker呢？有办法，就是装虚拟机，但是我一般不赞成装很重的虚拟机，就简单的就行了，所以我肯定是推荐使用`docker-toolbox`，下载的话推荐使用阿里云的镜像[docker-toolbox](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox)。
PS：遇到的一些坑
* 点击`Docker Quickstart Terminal`图标启动，不能正常启动

>因为会检测boot2docker是否是最新版本，所以很慢。所以先从Github上下载最新版本的[boot2docker](https://github.com/boot2docker/boot2docker/releases)，并复制到`C:\Users\Administrator\.docker\machine\cache`目录下。

* VM无法启动

>可能是window系统默认没有开动虚拟支持，所有先打开计算机的虚拟支持。进入`bios`寻找virtual ....>Disabled，修改成Enabled。

* 如何设置阿里云镜像加速

> ```bash
docker-machine create --engine-registry-mirror=https://f224s644.mirror.aliyuncs.com -d virtualbox default
```，其中`https://f224s644.mirror.aliyuncs.com`是我自己的阿里云专属镜像地址，自己使用的话，需要申请就行了[阿里云开发者平台](https://dev.aliyun.com)

至此，安装已经完成！
!["toolbox"](/images/docker_3.jpeg)

