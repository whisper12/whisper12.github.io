---
layout: post
title: jenkins的安装与实践
date: 2022-12-25 00:00:00.000000000 +09:00
---

>本篇以node构建的web项目为例子来进行一个自动化部署的任务配置

>首先介绍下整体思路、就是让jenkins获取拉去远程仓库的权限、进行打包压缩、传输到web服务器、删除或备份现有文件、解压更新文件、删除压缩包。

#### 安装环境

阿里云ECS centOs7

### 前提必要
安装好git 、node、以及源码仓库地址

### 安装JAVA环境
首先需要在服务器上安装 Java，因为 Jenkins 是用 Java 编写的。

```code
$ sudo yum update
$ sudo yum install java-11-openjdk
```

通过
```code
$ java --version
```
来查看JAVA是否安装成功
```code
openjdk 11.0.17 2022-10-18 LTS
OpenJDK Runtime Environment (Red_Hat-11.0.17.0.8-2.el7_9) (build 11.0.17+8-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-11.0.17.0.8-2.el7_9) (build 11.0.17+8-LTS, mixed mode, sharing)
```

### 安装jenkins

首先要在 /etc/apt/sources.list.d 目录中添加 Jenkins 的源
```code
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

添加 Jenkins 的 GPG 密钥

```code
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

接下来，可以使用下面的命令安装 Jenkins

```code
$ sudo yum update
$ sudo yum install jenkins
```

设置自动启动jenkins
```code
$ sudo systemctl enable jenkins
```

启动jenkins
```code
$ sudo systemctl start jenkins
```

这样jenkins已经安装完成并启动了，你可以通过访问 http://你的服务器地址:8080 来访问 Jenkins 的管理界面。

在管理界面中，你可以使用初始密码登录 Jenkins。初始密码可以在 /var/lib/jenkins/secrets/initialAdminPassword 文件中找到。你可以使用 cat 命令查看该文件的内容：

```code
cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 安装插件

在访问jenkins地址成功登录之后，选择默认安装插件即可

### 建立第一个自动化部署任务,这里以github仓库、自动部署一个node、webpack打包发布到web服务器为例子

首先需要在Jenkins上安装 `NodeJS Plugin` 和 `Publish Over SSH`两个插件

如果jenkins服务器与web服务器在同一个服务器上的话，可以不安装`Publish Over SSH`插件

首先进入菜单Dashboard>系统管理>插件管理

左边菜单栏选择Available plugins，搜索插件进行安装,可以勾选上安装完自动重启Jenkins选项

接下来需要将jenkins服务器上的git公钥添加到GitHub上的个人主页里的ssh里

`完成之后我们进入系统管理页面来配置Web服务器的账号，如和jenkins在同一个服务器和跳过`

![配置WEB服务器](/assets/images/2022-12-15-1.png)


### 配置完成之后，我们来构建第一个任务

新建任务，选择第一个free style即可

完成之后我们主要配置里面4个模块`源码管理、构建环境、Build Steps、构建后操作`

`源码管理`里添加源码的仓库地址
`构建环境`里选择`Provide Node & npm bin/ folder to PATH` 也就是node环境
`Build Steps`构建步骤
```code
npm i
npm run build
cd dist/build
tar zcvf web.tar.gz h5
```
我这里的打包步骤是安装依赖、开始构建、进入构建后目录、压缩构建目录，具体步骤看个人项目构建步骤来自定义

`构建后操作`
![构建后操作](/assets/images/2022-12-15-2.png)

```code
cd 页面目录
rm -rf h5
tar -xzvf web.tar.gz
rm -rf web.tar.gz
```

这样一个node的web页面自动化部署就完成了、可以点击立即构建开始尝试第一次构建