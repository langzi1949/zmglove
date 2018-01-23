---
title: Docker学习笔记--Web应用
date: 2018-01-12 18:39:56
tags:
    - Docker
---
!["Docker"](/images/docker.jpg)
本次主要是学习一下如何部署一个Web应用，并且多个容器之间进行交互。

#### Python-Flask
使用之前的`python-flask`的方式，进行一个Web项目的部署：
```python
# coding = utf-8
from flask import Flask,Response
import requests
import hashlib

app = Flask(__name__)

default_name = 'ZMGLOVE Blog'

salt = 'UNIQUE_SALT'

@app.route('/',methods=['GET','POST'])
def mainpage():
    
    name = default_name

    if request.method =='POST':
        name = request.form['name']
    
    salted_name = salt + name
    name_hash = hashlib.sha256(salted_name.encode()).hexdigest()

    header = '<html><head><title>Hello_World</title></head><body>'
    body = '''<form method="POST">
            Hello <input type="text" name="name" value="{0}">
            <input type="submit" value="submit">
            </form>
            <p>You look like a:
            <img src="/monster/{1}"/>
            '''.format(name,name_hash)
    footer = '</body></html>'

    return header + body + footer

#获取到图片
@app.route('/monster/<name>')
def get_identicon(name):
    r = requests.get('http://dnmonster:8080/monster/' + name + '?size=80')
    image = r.content
    
    return  Response(image,mimetype='image/png')

if __name__ == '__main__':
    # 这边使用0.0.0.0，因为需要把它绑定到所有网络接口，否则无法让主机或者其他容器访问到
    app.run(debug=True,host='0.0.0.0')
```
<!--more-->
其中这个里面使用了`hashlib`对姓名进行加密处理。
还要修改Dockerfile文件
```bash
RUN pip install Flask==0.10.1 uWSGI==2.0.8 requests==2.5.1 redis==2.10.3
```

#### 利用现有镜像-dnmonster
使用`dnmonster`的`identicon`的服务进行图片的渲染。

* 先运行`dnmonster`
```bash
sudo docker run -d --name dnmonster amouat/dnmonster:1.0
```
* 使用`docker --link`进行容器之间的连接
```bash
sudo docker run -d -p 5000:5000 -e "ENV=DEV" --link dnmonster:dnmonster helloflask
```
访问页面，就会出现以下效果：
!["Docker_5"](/images/docker_5.jpg)

#### 缓存-Redis
一般应用都会添加redis，用于缓存一些数据，使得I/O效率更高，性能更好！

* 首先修改python代码
```python
import redis
cache = redis.StrictRedis(host='redis',port=6379,db=0)

#获取到图片
@app.route('/monster/<name>')
def get_identicon(name):

    #判断redis中是否存在
    image = cache.get(name)
    if image is None:
        print("Cache miss ,flush=True")
        r = requests.get('http://dnmonster:8080/monster/' + name + '?size=80')
        image = r.content
        cache.set(name,image)
    
    return  Response(image,mimetype='image/png')
```

* 运行Redis容器
```bash
sudo docker run -d --name redis redis:3.0
```

* `docker run --link`连接redis

#### docker-compose
一般都是使用`docker-compose`进行自动化部署，不需要每次都要手动输入命令，直接运行就行。
```bash
helloflask:
    build: .
    ports:
      - "5000:5000"
    environment:
      ENV: DEV
    volumes:
      - ./app:/app
    links:
      - dnmonster
      - redis
dnmonster:
    image: amouat/dnmonster:1.0
redis:
    image: redis:3.0
```
最终运行`sudo docker-compose up -d`就行了！！