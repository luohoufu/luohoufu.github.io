---
title: 在centos 7下源码编译redis 3.2.1
date: 2016-07-07 19:30:49
tags: redis
---

## 准备工作
由于redis测试依赖tcl，在源码编译前先安装tcl
``` bash
[root@snails ~]# yum -y install tcl
```
## 下载redis源码
``` bash
[root@snails ~]# git clone https://github.com/antirez/redis.git
```
## 编译、测试、安装
``` bash
[root@snails ~]#  cd redis
[root@snails redis]#  make
[root@snails redis]#  make test
[root@snails redis]#  make install
``` 
* 查看安装结果
``` bash
[root@snails redis]#  ll /usr/local/bin/redis-*
```
## 配置文件
* 由于使用的是最新的redis 3.2.1版本，不存在[安全漏洞](https://help.aliyun.com/knowledge_detail/5988808.html)，默认bind 127.0.0.1
* 复制配置文件，并修改为后台运行 daemonize yes
``` bash
[root@snails redis]#  cp redis.conf /etc/redis_6379.conf
[root@snails redis]# vi /etc/redis_6379.conf
daemonize yes
```
## 启动redis
``` bash
redis-server /etc/redis_6379.conf
```
## 客户端测试（本机）
``` bash
[root@snails redis]# redis-cli 
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```
