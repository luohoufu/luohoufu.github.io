---
title: 在centos 7下源码编译golang 1.7
date: 2016-07-07 18:57:23
tags: golang
---

## 准备工作
* 一个“干净”的系统是必须的，本次操作在阿里云上完成。
``` bash
[root@snails ~]#hostnamectl set-hostname  snails
[root@snails ~]# hostnamectl
Static hostname: snails
Icon name: computer-vm
Chassis: vm
Machine ID: 7d26c16f116042a684ea498c9e2c240f
Boot ID: e567275688e84ce3b72a11794dc8ac9b
Virtualization: xen
Operating System: CentOS Linux 7 (Core)
CPE OS Name: cpe:/o:centos:centos:7
Kernel: Linux 3.10.0-327.el7.x86_64
Architecture: x86-64
```
## 实践过程
### 配置yum源
[CentOS 7配置](http://mirrors.aliyun.com/help/centos)
``` bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
#### 安装git、gcc、vim
``` bash
yum -y install git gcc vim
[root@snails ~]# git --version
git version 1.8.3.1
[root@snails src]# gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-4)
Copyright © 2015 Free Software Foundation, Inc.
本程序是自由软件；请参看源代码的版权声明。本软件没有任何担保；
包括没有适销性和某一专用目的下的适用性担保。
```
#### 下载go 1.4分支
``` bash
[root@snails ~]#git clone -b release-branch.go1.4  https://github.com/golang/go.git go
```
### 编译并配置环境变量
* 编译
``` bash
[root@snails ~]#cd go/src
[root@snails ~]#./all.bash
ALL TESTS PASSED
Installed Go for linux/amd64 in /root/go
Installed commands in /root/go/bin
*** You need to add /root/go/bin to your PATH.
```
* 配置环境变量
``` bash
[root@snails ~]#cd ~ && mkdir -p golang/{src,pkg,bin}
[root@snails ~]#vi /etc/profile
export GOPATH=$HOME/golang
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
[root@snails ~]# source /etc/profile
[root@snails ~]# go version
go version go1.4.3 linux/amd64
```
### 更新go版本再次编译
* 更新go版本
``` bash
[root@snails ~]# mv go go-bootstrap
[root@snails ~]# git clone https://github.com/golang/go.git
[root@snails ~]# cd go
``` 
* 再次编译
``` bash
[root@snails go]# vi /etc/profile
export GOROOT_BOOTSTRAP=$HOME/go-bootstrap
[root@snails go]# source /etc/profile
[root@snails go]# cd src/
[root@snails src]# ./clean.bash
[root@snails src]# ./all.bash
``` 
## 实践验证
``` bash
[root@snails src]# go version
go version devel +d872201 Thu Jul 7 04:06:52 2016 +0000 linux/amd64
```
