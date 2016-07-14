---
title: 在CentOS7上用systemctl配置tomcat 8
date: 2016-07-14 14:39:50
tags: java
---

## 环境准备

### 安装java环境
``` bash
[root@snails ~]# yum -y install java
[root@snails local]# java -version
openjdk version "1.8.0_91"
OpenJDK Runtime Environment (build 1.8.0_91-b14)
OpenJDK 64-Bit Server VM (build 25.91-b14, mixed mode)
```

### 下载tomcat并解压
``` bash
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.0.36/bin/apache-tomcat-8.0.36.tar.gz -P /usr/local
[root@snails local]# cd /usr/local/
[root@snails local]# tar -zxvf apache-tomcat-8.0.36.tar.gz > /dev/nul
[root@snails local]# ln -s /usr/local/apache-tomcat-8.0.36 /usr/local/tomcat
[root@snails local]# rm -rvf apache-tomcat-8.0.36.tar.gz 
已删除"apache-tomcat-8.0.36.tar.gz"
```
### 设置环境变量
``` bash
[root@snails local]# vi /etc/profile
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
export CLASS_PATH=.:$JAVA_HOME/lib
export CATALINA_HOME=/usr/local/tomcat
export CATALINA_BASE=/usr/local/tomcat
export PATH=$PATH:$JAVA_HOME/bin:$CATLINA_HOME:/bin
[root@snails ~]# source /etc/profile
```

>centos7 使用 systemctl 替换了 service命令

``` bash
[root@snails ~]# systemctl list-unit-files --type service  #查看全部服务命令
[root@snails ~]# systemctl status name.service #查看服务命令
[root@snails ~]# systemctl start name.service  #启动服务
[root@snails ~]# systemctl stop name.service  #停止服务
[root@snails ~]# systemctl restart name.service  #重启服务
[root@snails ~]# systemctl enable name.service  #增加开机启动
[root@snails ~]# systemctl disable name.service  #删除开机启动
```
> tips: 其中.service 可以省略。

## tomcat增加启动参数
tomcat 需要增加一个pid文件

>在tomca/bin 目录下面，增加 **setenv.sh** 配置，catalina.sh启动的时候会调用，同时配置java内存参数。

``` bash
[root@snails ~]# vi /usr/local/tomcat/bin/setenv.sh
#add tomcat pid
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
#add java opts
JAVA_OPTS="-server -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=1024m -Xms512M -Xmx1024M -XX:MaxNewSize=256m"
```

## 创建tomcat启动用户并授权

``` bash
[root@snails ~]# getent group tomcat || groupadd -r tomcat
[root@snails ~]# getent passwd tomcat || useradd -r -d /opt -s /bin/nologin -g tomcat tomcat
[root@snails ~]# chown -R tomcat:tomcat /usr/local/apache-tomcat-8.0.36
```

## 增加systemd-tomcat.service
在/usr/lib/systemd/system目录下增加systemd-tomcat.service，目录必须是绝对目录。
``` bash
[root@snails ~]# cd /usr/lib/systemd/system
[root@snails ~]# cat >systemd-tomcat.service <<EOF
[Unit]
Description=Apache Tomcat 8
After=syslog.target network.target
 
[Service]
Type=forking
PIDFile=/usr/local/tomcat/tomcat.pid
ExecStart=/usr/local/tomcat/bin/startup.sh 
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
User=tomcat
Group=tomcat 

[Install]
WantedBy=multi-user.target
EOF
```

## 使用systemd-tomcat.service
``` bash
[root@snails ~]# systemctl enable systemd-tomcat
[root@snails ~]# systemctl start systemd-tomcat
[root@snails ~]# ps aux |grep tomcat
tomcat    1304 34.8  2.8 3570504 102908 ?      Sl   14:36   0:03 /usr/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=1024m -Xms512M -Xmx1024M -XX:MaxNewSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=/usr/local/tomcat/endorsed -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
[root@snails system]# curl -I localhost:8080
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 14 Jul 2016 06:39:13 GMT
```

>因为配置pid，在启动的时候会再tomcat根目录生成tomcat.pid文件，停止之后删除。
同时tomcat在启动时候，执行start不会启动两个tomcat，保证始终只有一个tomcat服务在运行。
