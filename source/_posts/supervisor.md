---
title: 在CentOS7上用supervisor运行golang守护进程
date: 2016-07-14 11:53:22
tags: golang
---

## 安装pip
###下载pip安装文件并执行安装
* 下载文件
``` bash
[root@snails ~]# wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate
```
* 执行安装
``` bash
[root@snails ~]# python get-pip.py
[root@snails ~]# pip -V
pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)
```
### 设置pip国内aliyun源
* 创建或修改配置文件
``` java
linux ~/.pip/pip.conf
windows %HOMEPATH%\pip\pip.ini
```
* 修改内容
``` java  
[global]  
index-url = http://mirrors.aliyun.com/pypi/simple/  

  [install]
trusted-host=mirrors.aliyun.com  
```
* 更新pip到最新版本
``` bash
[root@snails ~]# pip install -U pip
```
* 查看已安装的库
``` bash
[root@snails ~]# pip list
```
## 安装supervisor
### 安装
``` bash
[root@snails ~]# pip install supervisor
```
  安装成功便可以拥有Supervisor，如果没有启动脚本，可以从 [这里](https://github.com/cedricporter/supervisor_conf/blob/master/init.d/supervisor) 下载一份，放置到  /usr/lib/systemd/system 或 /etc/systemd/system 目录（后者优先级更高）下面便可。
``` bash
[root@snails ~]# wget https://raw.githubusercontent.com/Supervisor/initscripts/master/centos-systemd-etcs -O /usr/lib/systemd/system/systemd-supervisor.service
```
### 配置
通过Supervisor附送的贴心的小脚本生成默认的配置文件
``` bash
[root@snails ~]# echo_supervisord_conf > /etc/supervisord.conf
```
　我们可以根据需要修改里面的配置。我这里，每个不同的项目，使用了一个单独的配置的文件，放置在 /etc/supervisor/下面，于是修改 /etc/supervisord.conf ，加上如下内容：
``` bash
[include]
files = /etc/supervisor/*.conf
```
## 创建golang http服务
为了测试方便，我这里用一个最简单的golang http服务。
``` bash
[root@snails ~]# vi ~/simple_http_server.go
```
``` go
package main

 import (
	"fmt"
	"log"
	"net/http"
)

 func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello world\n")
	})

	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```
直接运行这个程序会占用住终端，下面看看如何用supervisor来跑这个程序。
### 创建golang服务对应的supervisor配置文件
``` bash
vi /etc/supervisor/golang.conf
[program:golang-http-server]
command=/root/simple_http_server
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/simple_http_server.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/simple_http_server.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
```
* 几个配置说明：
``` bash
command：表示运行的命令，填入完整的路径即可。autostart：表示是否跟随supervisor一起启动。autorestart：如果该程序挂了，是否重新启动。stdout_logfile：终端标准输出重定向文件。stderr_logfile：终端错误输出重定向文件。
```
## 启动supervisor
``` bash
[root@snails ~]# /usr/bin/supervisord -c /etc/supervisord.conf
```
如果出现什么问题，可以查看日志进行分析，日志文件路径/tmp/supervisord.log
> tips：如果修改了配置文件，可以用kill -HUP重新加载配置文件

  ``` bash
[root@snails ~]# cat /tmp/supervisord.pid | xargs sudo kill -HUP
```
### 查看supervisor运行状态
  ``` bash
[root@snails ~]# supervisorctl
golang-http-server               RUNNING   pid 30343, uptime 0:00:55
```
  * 输入help可以查看帮助
``` bash
supervisor> help
default commands (type help <topic>):
=====================================
add    exit      open  reload  restart   start   tail   
avail  fg        pid   remove  shutdown  status  update 
clear  maintail  quit  reread  signal    stop    version
```
## supervisor运行原理
supervisor运行后本身是守护进程，通过自身来管理相应的子进程，通过观察相应的进程状态就很明了。
``` bash
[root@snails ~]# ps -ef | grep supervisord
root     30269     1  0 11:31 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c /etc/supervisord.conf
[root@snails ~]# ps -ef | grep simple_http_server
root     30343 30269  0 11:45 ?        00:00:00 /root/simple_http_server
```
> 可以很直观的看出golang simple_http_server进程是supervisord的子进程。

 ## supervisor是否靠谱
supervisor的诞生已经10年了，现在是3+版本，所以放心使用吧。
