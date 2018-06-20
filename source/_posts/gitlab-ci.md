---
title: GitLab-CI实战
date: 2018-06-18 08:09:23
tags:
    - 运维
---
### 简介
从 GitLab 8.0 开始，GitLab CI 就已经集成在 GitLab 中，我们只要在项目中添加一个 `.gitlab-ci.yml` 文件，然后添加一个 Runner，即可进行持续集成。本文将介绍如何使用 GitLab CI 进行持续集成。

### 常见名词

##### Pipeline
一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。
任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：
```bash
+------------------+           +----------------+
|                  |  trigger  |                |
|   Commit / MR    +---------->+    Pipeline    |
|                  |           |                |
+------------------+           +----------------+
```
##### Stages
Stages 表示构建阶段，说白了就是上面提到的流程。
我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：
<!--more-->

* 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
* 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
* 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

因此，Stages 和 Pipeline 的关系就是：
```bash
+--------------------------------------------------------+
|                                                        |
|  Pipeline                                              |
|                                                        |
|  +-----------+     +------------+      +------------+  |
|  |  Stage 1  |---->|   Stage 2  |----->|   Stage 3  |  |
|  +-----------+     +------------+      +------------+  |
|                                                        |
+--------------------------------------------------------+
```

### 安装Gitlab-Runner
>理解了上面的基本概念之后，有没有觉得少了些什么东西 —— 由谁来执行这些构建任务呢？
答案就是 GitLab Runner 了！

想问为什么不是 GitLab CI 来运行那些构建任务？
一般来说，构建任务都会占用很多的系统资源 (譬如编译代码)，而 GitLab CI 又是 GitLab 的一部分，如果由 GitLab CI 来运行构建任务的话，在执行构建任务的时候，GitLab 的性能会大幅下降。

GitLab CI 最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给 GitLab Runner 来做拉！
因为 GitLab Runner 可以安装到不同的机器上，所以在构建任务运行期间并不会影响到 GitLab 的性能~

#### 安装步骤
参考[Gitlab官方文档](https://docs.gitlab.com/runner/install/linux-repository.html)
```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
sudo yum install gitlab-ci-multi-runner
```
#### 注册Runner
安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 CI 绑定就可以了：
* 打开你 GitLab 中的项目页面，在项目设置中找到 runners
* 运行 `sudo gitlab-ci-multi-runner register`
* 输入 CI URL
* 输入 Token
* 输入 Runner 的名字
* 选择 Runner 的类型，简单起见还是选 Shell 吧
* 完成

当注册好 Runner 之后，可以用 `sudo gitlab-ci-multi-runner list` 命令来查看各个 Runner 的状态：
```bash
sudo gitlab-runner list
```

### .gitlab-ci.yml文件
配置好 Runner 之后，我们要做的事情就是在项目根目录中添加 `.gitlab-ci.yml` 文件了。
当我们添加了 `.gitlab-ci.yml` 文件后，每次提交代码或者合并 MR 都会自动运行构建任务了。
这边给一个事例：
```yml
# version:1.0
# update: 2018-06-07
# from chenzehao@hidotech.com

### 测试环境
.testRunner: &testRunner
  variables:
    SERVER_NAME: "testmgh"
    SERVER_PORT: "22222"
    DEST: "/mushroom/server/java/"
    PROGRAM_NAME: "mm-merchant-service"



##### 警告：以下内容请勿更改！！！ #####
before_script:
  - echo ${SERVER_NAME:?'no data'}
  - echo ${SERVER_PORT:?'no data'}
  - echo ${DEST:?'no data'}
  - echo ${PROGRAM_NAME:?'no data'}
  - \[ "/" = $DEST \] && { echo '$DEST not allowed!';exit 1; }
  - \[ ! "/" = ${DEST:$((${#DEST}))-1} \] && { echo '$DEST 请使用/结尾';exit 1; }
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
  - eval $(ssh-agent -s)
  
  

stages:
  - test

test:
  stage: test
  script:
    - whoami
    - ssh-add <(echo "$test_key")
    - echo $SERVER_NAME
    - mvn clean package -Ptest -Dmaven.test.skip=true
    - ls -al target/$CI_PROJECT_NAME.jar && md5sum target/$CI_PROJECT_NAME.jar
    - ssh -p$SERVER_PORT root@$SERVER_NAME "supervisorctl stop java:$PROGRAM_NAME"
    - ssh -p$SERVER_PORT root@$SERVER_NAME "find $DEST -maxdepth 1 -type d -name $CI_PROJECT_NAME | xargs rm -rf"
    - scp -P$SERVER_PORT target/$CI_PROJECT_NAME.jar root@$SERVER_NAME:$DEST
    - ssh -p$SERVER_PORT root@$SERVER_NAME "md5sum $DEST$CI_PROJECT_NAME.jar"
    - ssh -p$SERVER_PORT root@$SERVER_NAME "supervisorctl start java:$PROGRAM_NAME"
    - ssh -p$SERVER_PORT root@$SERVER_NAME "supervisorctl status java:"
  <<: *testRunner
  only:
    - /^release.*$/
    - /^hotfix.*$/
  tags:
    - local
  when: manual
```

### 参考
[用 GitLab CI 进行持续集成](https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)
[gitlab之gitlab-ci自动部署](https://www.jianshu.com/p/df433633816b)
[gitlab-ci踩过的坑](https://www.cnblogs.com/restful/p/6711063.html)






