---
title: 在CentOS7上源码安装OpenResty
date: 2016-07-08 14:05:04
tags: openresty
---

## 环境准备
您必须将这些库 
perl 5.6.1+
libreadline
libpcre
libssl
安装在您的电脑之中。 对于 Linux来说, 您需要确认使用 ldconfig 命令，让其在您的系统环境路径中能找到它们。
* CentOS 7 安装OpenResty所需依赖
``` bash
[root@snails ~]# yum -y install readline-devel pcre-devel openssl-devel gcc
```
## 构建 OpenResty
### 下载
从下载页 [Download](http://openresty.org/cn/download.html)下载最新的ngx_openresty源码包，并且像下面的示例一样将其解压:
``` bash
[root@snails ~]# wget https://openresty.org/download/openresty-VERSION.tar.gz
[root@snails ~]# tar xzvf ngx_openresty-VERSION.tar.gz
```
VERSION  的地方替换成您下载的源码包的版本号，比如说 1.9.15.1。

### 编译安装
然后在进入 ngx_openresty-VERSION/
 目录, 然后输入以下命令配置:
./configure --prefix=/root/openresty

默认, --prefix=/usr/local/openresty
 程序会被安装到/usr/local/openresty目录。
您可以指定各种选项，比如
``` bash
[root@snails openresty-1.9.15.1]# ./configure --prefix=/opt/openresty \
                                              --with-luajit \
                                              --with-http_stub_status_module
gmake
gmake install
```
试着使用 ./configure --help 查看更多的选项。
### 设置环境变量及文件软链接
``` bash
[root@snails ~]# ln -s /usr/local/openresty/nginx /usr/local/nginx
[root@snails ~]# vi /etc/profile
export ORPATH=/usr/local/openresty
export PATH=$PATH:$ORPATH/bin:$ORPATH/nginx/sbin
```
### 配置用户及组
``` bash
[root@snails nginx]# groupadd -f www  
[root@snails nginx]# useradd -s /sbin/nologin -g www www
[root@snails nginx]# vi conf/nginx.conf
user www www;
```
## 验证
``` bash
[root@snails nginx]# nginx
[root@snails nginx]# curl -I localhost
HTTP/1.1 200 OK
```
