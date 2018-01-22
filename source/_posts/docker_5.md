---
title: Docker学习笔记--Hello World
date: 2018-01-03 13:08:15
tags:
    - Docker
---
!["Docker"](/images/docker.jpg)
今天我们开始第一个Web应用，使用Docker容器进行部署，So，Let's Go!!!

#### Hello World
首先创建一个app的目录，在app下面创建python文`hello_world.py`
```python
# coding = utf-8
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    # 这边使用0.0.0.0，因为需要把它绑定到所有网络接口，否则无法让主机或者其他容器访问到
    app.run(debug=True,host='0.0.0.0')
```
然后在app里面再创建Dockerfile文件：
<!--more-->
```bash
FROM python:3.4

RUN pip install Flask==0.10.1
WORKDIR /app
COPY app /app

CMD ["python","hello_world.py"]
```

#### 绑定挂载
以上的方法有个很大的缺陷，就是修改代码都需要重新创建镜像，这样很麻烦。但是有个办法可以解决：`绑定挂载`。
前提条件，先停掉容器，并删除容器。
```bash
# 可以通过sudo docker ps查看容器运行的id
sudo docker stop $(docker ps -lq)
sudo rm $(docker ps -lq)
#  -v "$PWD"/app:/app 参数把位于 /app 的 app 目录挂载到容器内
sudo docker run -d -p 5000:5000 -v "$PWD"/app:/app helloflask  #helloflask 是镜像的名称
```
然后修改主机上的`hello_world.py`，会立马生效！！

#### uWSGI
前面的方法，其实是Flask自带的Web容器，现在使用常见的生产环境的应用服务器：`uWSGI`。
修改`Dockerfile`文件：
```bash
FROM python:3.4

RUN pip install Flask==0.10.1 uWSGI==2.0.8
WORKDIR /app
COPY app /app

CMD ["uwsgi","--http","0.0.0.0:5000","--wsgi-file","/app/hello_world.py","--callable","app","--stats","0.0.0.0:5151"]
```

>这里我们告诉 uWSGI 启动一个监听 5000 端口的
HTTP 服务器， 并从 /app/identidock.py 运行 app 应用。 它还在 5151 端口启动一个数据
统计服务器。其实我们还可以在运行 docker run 命令时，重新定义 CMD 的内容。

但是上面的方法有个安全问题，服务器是在root权限执行的，就是一旦入侵到容器中，就可以获得root权限，这个是相当危险的。可以指定运行服务器的用户：
```bash
FROM python:3.4

RUN groupadd -r uwsgi && useradd -r -g uwsgi uwsgi  # 这个是添加用户组/用户
RUN pip install Flask==0.10.1 uWSGI==2.0.8
WORKDIR /app
COPY app /app

EXPOSE 5000 5151  # 指明主机和容器可以访问的端口
USER uwsgi  # 以uwsgi用户执行以下命令

CMD ["uwsgi","--http","0.0.0.0:5000","--wsgi-file","/app/hello_world.py","--callable","app","--stats","0.0.0.0:5151"]
```
然后`sudo docker port CONTAINER_NAME` 就可以查看端口的映射（让Docker自行寻找端口，比自己分配端口更加轻松）

#### 开发环境和生产环境
一般都是分环境的进行部署的，分为`开发环境，测试环境，生产环境`等
首先，创建一个脚本文件cmd.sh:
```bash
#!/bin/bash
set -e

if [ "$ENV" == 'PROD' ]; then
        echo "Running Production Server"
        exec uwsgi --http 0.0.0.0:5000 --wsgi-file /app/hello_world.py --callable app --stats 0.0.0.0:5151
else
        echo "Running Development Server"
        exec python "hello_world.py"

fi
```
然后，修改Dockerfile文件：
```bash
FROM python:3.4

RUN groupadd -r uwsgi && useradd -r -g uwsgi uwsgi
RUN pip install Flask==0.10.1 uWSGI==2.0.8
WORKDIR /app
COPY app /app
COPY cmd.sh /

EXPOSE 5000 5151
USER uwsgi

CMD ["/cmd.sh"]
```
接着，修改`cmd.sh`的权限
```bash
sudo chomd +x cmd.sh
```
最后，创建镜像，并执行:
```bash
sudo docker build -t helloflask .
sudo docker run -e "ENV=DEV" -p 5000:5000 helloflask

Running Development Server
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 130-199-939
```