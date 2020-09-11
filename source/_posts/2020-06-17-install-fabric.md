---
title: 安装Hyperledger Fabric记录
tags:
  - 区块链
  - Hyperledger Fabric
  - Linux
date: 2020-06-17 19:23:40
categories: 区块链
---

写这篇文章主要是记录下自己在搭建Hyperledger Fabric时候遇到的一些坑。做个备忘吧。

<!-- more -->

**搭建环境**：Ubuntu 18.04 LTS系统
**参考**：[Hyperledger Fabric中文文档](https://hyperledger-fabric.readthedocs.io/zh_CN/latest/install.html)

# 环境准备
在官方文档中，需要一些所谓的“先决条件”，主要需要的环境有如下这些：

软件 | 版本
 --  | -- 
cURL | 最新版
wget | 最新版
Docker | 17.06.2-ce 及以上
Docker-compose |  1.14.0 或更高版本
Go编程语言 | 1.13.x 版本（我实际安装了1.14.2）

## 安装cURL
需要安装最新版本，执行命令
```bash
$ sudo apt update
$ sudo apt install curl
```
实在不行的话，可以下载编译源码进行安装[cURL下载地址](https://curl.haxx.se/download/)

## 安装wget
只有如果使用官方提供的脚本进行安装，则`wget`将用于下载源码，用`apt install`即可安装

## 安装docker
参考文档：[docker的Ubuntu安装文档](https://docs.docker.com/engine/install/ubuntu/)

### 卸载旧版本
旧版本中，docker被称为`docker`,`docker.io`,`docker-engine`等，卸载她们
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```
若`apt`报告说这些包没有被安装，那么就说明没有旧的版本。

### 通过添加仓库安装docker
这是比较方便的方法，所以我选这种方式安装
#### 设置仓库（repository）
1. 更新`apt`包索引，安装下面这些包使得`apt`能够通过HTTPS使用仓库
```bash
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

2. 添加Docker的官方GPG密钥：
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
确认指纹为`9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88`,如下所示：
```bash
$ sudo apt-key fingerprint 0EBFCD88

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

3. 设置使用的仓库版本。因为我使用的是Linux amd64的系统，所以我使用如下设置，若有不同，参考[docker的Ubuntu安装文档](https://docs.docker.com/engine/install/ubuntu/)
```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

#### 安装docker
更新`apt`包索引后，就可以通过`apt`直接安装了：
```bash
 $ sudo apt-get update
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
### 后续操作
设置docker daemon 在系统启动的时候自动启动，使用下边的命令：
```bash
$ sudo systemctl enable docker
```
将自己的用户添加到 docker 组，这样就不用每次执行docker都使用`sudo`了。
```bash
$ sudo usermod -a -G docker <username>
```

## 安装docker compose
参考：[compose docs](https://docs.docker.com/compose)

1. 下载docker-compose
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
2. 给予文件夹可执行权限
```bash
$ sudo chmod +x /usr/local/bin/docker-compose
```
3. 如果执行`docker-compose`失败，则建立软连接
```bash
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
查看版本
```bash
$ docker-compose --version
docker-compose version 1.26.0, build d4451659
```

## 安装Go语言
下载Go语言包之后,解压到`/usr/local`，或者其他地方
```bash
$ sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```
把`/usr/local/go/bin`添加到环境变量，在`/etc/profile` (为整个系统安装)或 `$HOME/.profile`（为当前用户安装）最后一行添加如下语句：
```bash
export PATH=$PATH:/usr/local/go/bin
```
**注意**：更改完profile文件后，不会立即生效，需要执行如`source $HOME/.profile`的命令才会生效。

另外，还需要将`GOPATH`添加进环境变量中


## Node.js 运行环境及 NPM

