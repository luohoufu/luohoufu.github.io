---
title: 在CentOS上编译安装MySQL 5.7.13步骤详解
date: 2016-07-18 11:38:37
tags: mysql
---

## MySQL 5.7主要特性
1. 更好的性能
    对于多核CPU、固态硬盘、锁有着更好的优化，每秒100W QPS已不再是MySQL的追求，下个版本能否上200W QPS才是用户更关心的。
* 更好的InnoDB存储引擎
* 更为健壮的复制功能
    复制带来了数据完全不丢失的方案，传统金融客户也可以选择使用。MySQL数据库。此外，GTID在线平滑升级也变得可能。
* 更好的优化器
    优化器代码重构的意义将在这个版本及以后的版本中带来巨大的改进，Oracle官方正在解决MySQL之前最大的难题。
* 原生JSON类型的支持
* 更好的地理信息服务支持
    InnoDB原生支持地理位置类型，支持GeoJSON，GeoHash特性
* 新增sys库
    以后这会是DBA访问最频繁的库MySQL 5.7

## 安装准备
### 安装依赖包
``` bash
[root@snails ~]# yum -y install gcc gcc-c++ ncurses ncurses-devel cmake bison
```
### 下载相应源码包
``` bash
[root@snails ~]# wget https://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
[root@snails ~]# wget http://cdn.mysql.com/Downloads/MySQL-5.7/mysql-5.7.13.tar.gz
```
> 也可以使用官方[下载链接](http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.9.tar.gz) 进行下载。

### 新建MySQL用户和用户组
``` bash
[root@snails ~]# groupadd -r mysql && useradd -r -g mysql -s /sbin/nologin -M mysql 
```
### 预编译
``` bash
[root@snails ~]# tar -zxvf boost_1_59_0.tar.gz
[root@snails data]# md5sum mysql-5.7.13.tar.gz 
8fab75dbcafcd1374d07796bff88ae00  mysql-5.7.13.tar.gz
[root@snails ~]# tar -zxvf mysql-5.7.13.tar.gz
[root@snails data]# mkdir -p /data/mysql
[root@snails data]# cd mysql-5.7.13
[root@snails data]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DWITH_BOOST=../boost_1_59_0 \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EMBEDDED_SERVER=1
```
### 编译安装
``` bash
[root@snails mysql-5.7.13]# make -j `grep processor /proc/cpuinfo | wc -l`
#编译很消耗系统资源，小内存可能编译通不过make install
[root@snails mysql-5.7.13]# make install
```
### 设置启动脚本，开机自启动
``` bash
[root@snails mysql-5.7.13]# ls -lrt /usr/local/mysql
[root@snails mysql-5.7.13]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@snails mysql-5.7.13]# chmod +x /etc/init.d/mysqld
[root@snails mysql-5.7.13]# systemctl enable mysqld
mysqld.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig mysqld on
```
### 配置文件
``` bash
/etc/my.cnf，仅供参考 
[root@snails mysql-5.7.13]# cat > /etc/my.cnf << EOF
[client]
port = 3306
socket = /dev/shm/mysql.sock
[mysqld]
port = 3306
socket = /dev/shm/mysql.sock
basedir = /usr/local/mysql
datadir = /data/mysql
pid-file = /data/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4
#skip-name-resolve
#skip-networking
back_log = 300
max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M
thread_cache_size = 8
query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 30
log_error = /data/mysql/mysql-error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /data/mysql/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp
#lower_case_table_names = 1
skip-external-locking
default_storage_engine = InnoDB
#default-storage-engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
interactive_timeout = 28800
wait_timeout = 28800
[mysqldump]
quick
max_allowed_packet = 16M
[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
EOF
```
### 添加mysql的环境变量
``` bash
[root@snails mysql-5.7.13]# echo -e '\n\nexport PATH=/usr/local/mysql/bin:$PATH\n' >> /etc/profile && source /etc/profile
```
### 初始化数据库
``` bash
[root@snails mysql-5.7.13]# mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql
```
>注：
- MySQL之前版本mysql_install_db是在mysql_basedir/script下
- MySQL 5.7直接放在了mysql_install_db/bin目录下。
- "–initialize"已废弃,生成一个随机密码(~/.mysql_secret)
- "–initialize-insecure"不会生成密码 
- "–datadir"目录下不能有数据文件

### 启动数据库 
``` bash
[root@snails mysql-5.7.13]# systemctl start mysqld
[root@snails mysql-5.7.13]# systemctl status mysqld
● mysqld.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/rc.d/init.d/mysqld)
   Active: active (running) since 一 2016-07-18 11:15:35 CST; 8s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 23927 ExecStart=/etc/rc.d/init.d/mysqld start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/mysqld.service
           ├─23940 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/mysql.pid
           └─24776 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/mysql-err...

7月 18 11:15:32 snails systemd[1]: Starting LSB: start and stop MySQL...
7月 18 11:15:35 snails mysqld[23927]: Starting MySQL..[  OK  ]
7月 18 11:15:35 snails systemd[1]: Started LSB: start and stop MySQL.
```
### 查看MySQL服务进程和端口
``` bash
[root@snails mysql-5.7.13]# ps -ef | grep mysql
root     23940     1  0 11:15 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/mysql.pid
mysql    24776 23940  0 11:15 ?        00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/mysql-error.log --open-files-limit=65535 --pid-file=/data/mysql/mysql.pid --socket=/dev/shm/mysql.sock --port=3306
[root@snails mysql-5.7.13]# netstat -tunpl | grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      24776/mysqld
```
### 设置数据库root用户密码
　　MySQL和Oracle数据库一样，数据库也默认自带了一个 root 用户（这个和当前Linux主机上的root用户是完全不搭边的），我们在设置好MySQL数据库的安全配置后初始化root用户的密码。配制过程中，一路输入 y 就行了。这里只说明下MySQL5.7.13版本中，用户密码策略分成低级 LOW 、中等 MEDIUM 和超强 STRONG 三种，推荐使用中等 MEDIUM 级别！
``` bash
[root@snails mysql-5.7.13]# mysql_secure_installation
```
## 常用操作
### 将MySQL数据库的动态链接库共享至系统链接库
　　一般MySQL数据库还会被类似于PHP等服务调用，所以我们需要将MySQL编译后的lib库文件添加至当前Linux主机链接库 /etc/ld.so.conf.d/
 下，这样MySQL服务就可以被其它服务调用了。
``` bash
 [root@snails mysql-5.7.13]# ldconfig |grep mysql
[root@snails mysql-5.7.13]# echo "/usr/local/mysql/lib" > /etc/ld.so.conf.d/mysql.conf
[root@snails mysql-5.7.13]# ldconfig
[root@snails mysql-5.7.13]# ldconfig -v |grep mysql
ldconfig: 无法对 /libx32 进行 stat 操作: 没有那个文件或目录
ldconfig: 多次给出路径“/usr/lib”
ldconfig: 多次给出路径“/usr/lib64”
ldconfig: 无法对 /usr/libx32 进行 stat 操作: 没有那个文件或目录
/usr/lib64/mysql:
	libmysqlclient.so.18 -> libmysqlclient.so.18.0.0
/usr/local/mysql/lib:
	libmysqlclient.so.20 -> libmysqlclient.so.20.3.0
```
### 创建其它MySQL数据库用户
``` bash
[root@snails mysql-5.7.13]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.13-log Source distribution

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```
``` mysql
mysql>CREATE DATABASE `tonnydb` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected (0.01 sec)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| tonnydb            |
+--------------------+
5 rows in set (0.00 sec)
mysql> grant all privileges on tonnydb.* to 'tonny@%' identified by 'Hi.Tonny@888';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```
