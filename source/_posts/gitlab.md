---
title: Gitlab安装
date: 2018-06-17 09:29:43
tags:
    - 运维
---
### 简介
Gitlab是代码的在线托管工具，能够实现版本控制。

### 安装步骤
以下操作都是基于*CentOS 7*

#### 1.安装一些依赖(在系统防火墙中打开 HTTP and SSH连接，邮件服务)
```bash
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld

sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```
<!--more-->
#### 2.添加Gitlab包
```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```
#### 3.修改访问Gitlab-Web的路径
```bash
sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ee
```
PS：以上的URL请按照自己的选择设置，也可以在`/etc/gitlab/gitlab.rb`文件中修改url
```bash
external_url 'http://gitlab.example.com'
```
最后需要加载配置，使其生效
```bash
sudo gitlab-ctl reconfigure
```

最后，就可以访问gitlab-web路径，记得第一次登录需要给root账号设置密码。

### 参考
[Gitlab官方文档](https://about.gitlab.com/installation/#centos-7)