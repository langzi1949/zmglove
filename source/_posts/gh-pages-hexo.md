---
title: gh-pages加上Hexo搭建Blog
date: 2017-08-24 16:40:36
tags: 
    - Hexo
---
关于这个教程,网上有很多的大神都已经教会大家怎么整了，我也不敢班门弄斧，并且也说不出什么花来，那我就直接上干货吧。
<br>

-----------------------------------
##### 大神博客
<a href="https://xuanwo.org/2015/03/26/hexo-intor/">史上最详细的Hexo教程</a>

-----------------------------------
#### 我在实际操作中遇见的问题

##### 1.执行npm install hexo-cli -g时运行卡顿
这个问题是因为npm的源出现了问题，至于原因嘛？大家都懂得！<br>
解决的办法是:<br>
1.通过config命令
``` bash
npm config set registry https://registry.npm.taobao.org
npm info underscore
```
2.命令行指定
``` bash
npm --registry https://registry.npm.taobao.org info underscore
```
3.编辑 ～／.npmrc  加入下面内容
``` bash
registry = https://registry.npm.taobao.org
```
上述三种方法都可以,但有时候不行,我是用的第三个。

##### 2.hexo deploy后github上显示的是打包后文件
这个问题是因为在github上只建了一个分支，而且分支取得名字是 gh-pages
其实gh-pages分支存放的都是deploy后的文件，而master分支应该是存放源码。
所以应该要建两个分支一个是*master*,另外一个是*gh-pages*。

##### 3.另外一台Mac，pull源码后执行hexo命令出错(local hexo not found...)
是因为remote后，pull下来的源码中没有node_modules目录,执行后会提醒你执行
*npm install hexo --save* ,这样是不行的,应该直接执行:
``` bash
npm install
```
PS:可能会有WARN 但是貌似没有问题

##### 4.hexo server后访问页面是一片空白
这个是因为themes目录中没有相对应的主题,我用的是Next,我已经在站配置文件中_config.yml
中配置了theme: next<br>
那现在就执行：
``` bash
cd themes
git clone https://github.com/iissnan/hexo-theme-next ./next/
```
这样整好以后,*hexo server* 就可以访问到页面内容了！
以上


