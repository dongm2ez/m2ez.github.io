---
layout: post
title: Ubuntu 搭建 GitLab 笔记
categories: development
tags: [git,gitlab]
---

## 简介

GitLab 社区版可以提供许多与 GitHub 相同的功能，且部署在属于自己的机器上，我们会因为网络及其他一些问题而不便使用 GitHub ，这时部署一个 GitLab 是最好的选择。

### 下载 GitLab 并安装

我的环境是 Ubuntu 16.04 下进行部署操作。

GitLab 下载地址：https://about.gitlab.com/downloads/#ubuntu1604

其他版本请自行选择不同系统。

1.首先是安装一些依赖服务

```
sudo apt-get install curl openssh-server ca-certificates postfix
```

2.官方的建议是使用脚本直接执行安装

```
sudo curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce
```

如果你能通过以上方式安装，恭喜你的网络很好，但一般因为大墙的存在这个方式很多时候并不能成功，所以我们要通过手动下载包的方式进行安装。

```
sudo curl -LJO https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/xenial/gitlab-ce-XXX.deb/download
sudo dpkg -i gitlab-ce-XXX.deb
```

以上是示例，具体版本需要进行替换，在 https://packages.gitlab.com/gitlab/gitlab-ce 中找到适合自己的 GitLab 版本，从 Download 获取到下载地址。

![](http://img.m2ez.com/14902391408842.jpg)

我选择的是目前最新的 GitLab 9.0 版本

使用 wget 或 curl 将这个包下载到服务器上。服务器下载慢的话可在本地用工具下载然后通过 SCP 或 ftp 传到服务器上去。

```
sudo dpkg -i gitlab-ce_9.0.0-ce.0_amd64.deb
```

使用以上命令进行安装。

打开/etc/gitlab/gitlab.rb,将external_url = 'http://git.example.com'修改为自己的域名地址：http://example.com，默认为80端口，如要使用其他端口后面加上端口号，如：http://127.0.0.1:8080。

然后执行：

```
sudo gitlab-ctl reconfigure
```

启动完成后浏览器访问配置好的地址，应该出现重置管理员密码的界面。

### 汉化

1.下载社区提供的汉化包，在 https://gitlab.com/xhang/gitlab/ 中找到相应的汉化分支。

```
sudo wget wget -cO gitlab-9.0_zh.tar.gz https://gitlab.com/xhang/gitlab/repository/archive.tar.gz?ref=9-0-stable-zh
```

2.解压包

```
sudo tar zxvf gitlab-9.0_zh.tar.gz
```

3.停止 GitLab 服务

```
sudo gitlab-ctl stop
```

4.备份 gitlab-rails 目录，该目录下主要是web应用部分，也是当前项目仓库的起始版本，也是汉化包要覆盖的目录。

```
sudo tar zcvf /opt/gitlab/embedded/service/gitlab-rails-bak.tar.gz gitlab-rails
```

5.将解压后的汉化补丁覆盖原来的

```
sudo cp -rf gitlab-9-0-stable-zh/* gitlab-rails/
```

6.启动服务

```
sudo gitlab-ctl start
```

7.重新执行配置命令

```
sudo gitlab-ctl reconfigure
```

汉化完成

### 一些界面设置

进入界面后关掉一些我们可能用不到的设置，在 「管理区域」的设置中进行更改

![](http://img.m2ez.com/14902593794119.jpg)

「开启 Gravatar 头像」关闭，国内访问不了，要想访问得翻墙
「开启注册」关闭，我们自己的仓库系统不需要公开注册，账号分配就好





