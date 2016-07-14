---
title: 在CentOS7上源码安装MongoDB 3.2.7
date: 2016-07-08 08:59:05
tags: mongodb
---

## 环境准备
``` bash
[root@snails ~]# mkdir /root/mongodb #创建MongoDB程序存放目录
[root@snails ~]#mkdir /data/mongodata -p #创建数据存放目录
[root@snails ~]# mkdir /data/log/mongolog -p #创建日志存放目录
[root@snails ~]# curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.2.7.tgz
```
## 安装
``` bash
[root@snails ~]# tar xf mongodb-linux-x86_64-3.2.7.tgz
[root@snails ~]# cd mongodb-linux-x86_64-3.2.7/
[root@snails ~]# cp -r * /root/mongodb
```
* 为了便于命令启动，需要编辑全局变量PATH
``` bash
[root@snails ~]# vi /etc/profile.d/mongo.sh
export PATH=$PATH:/root/mongodb/bin
[root@snails ~]# source /etc/profile
```
## 启动服务
 首先查看mongod的帮助信息

``` bash
[root@snails ~]# pwd
/root/mongodb/bin
[root@snails ~]# vi /etc/profile.d/mongo.sh
[root@snails ~]# source /etc/profile
[root@snails ~]# mongod --help
Options:

General options:
  -h [ --help ]                         show this usage information
  --version                             show version information
  -f [ --config ] arg                   configuration file specifying 
                                        additional options
  -v [ --verbose ] [=arg(=v)]           be more verbose (include multiple times
                                        for more verbosity e.g. -vvvvv)
  --quiet                               quieter output
  --port arg                            specify port number - 27017 by default
  --bind_ip arg                         comma separated list of ip addresses to
                                        listen on - all local ips by default
  --ipv6                                enable IPv6 support (disabled by 
                                        default)
  --maxConns arg                        max number of simultaneous connections 
                                        - 1000000 by default
  --logpath arg                         log file to send write to instead of 
                                        stdout - has to be a file, not 
                                        directory
  --syslog                              log to system's syslog facility instead
                                        of file or stdout
  --syslogFacility arg                  syslog facility used for mongodb syslog
                                        message
  --logappend                           append to logpath instead of 
                                        over-writing
  --logRotate arg                       set the log rotation behavior 
                                        (rename|reopen)
  --timeStampFormat arg                 Desired format for timestamps in log 
                                        messages. One of ctime, iso8601-utc or 
                                        iso8601-local
  --pidfilepath arg                     full path to pidfile (if not set, no 
                                        pidfile is created)
  --keyFile arg                         private key for cluster authentication
  --noauth                              run without security
  --setParameter arg                    Set a configurable parameter
  --httpinterface                       enable http interface
  --clusterAuthMode arg                 Authentication mode used for cluster 
                                        authentication. Alternatives are 
                                        (keyFile|sendKeyFile|sendX509|x509)
  --nounixsocket                        disable listening on unix sockets
  --unixSocketPrefix arg                alternative directory for UNIX domain 
                                        sockets (defaults to /tmp)
  --filePermissions arg                 permissions to set on UNIX domain 
                                        socket file - 0700 by default
  --fork                                fork server process
  --auth                                run with security
  --jsonp                               allow JSONP access via http (has 
                                        security implications)
  --rest                                turn on simple rest api
  --slowms arg (=100)                   value of slow for profile and console 
                                        log
  --profile arg                         0=off 1=slow, 2=all
  --cpu                                 periodically show cpu and iowait 
                                        utilization
  --sysinfo                             print some diagnostic system 
                                        information
  --noIndexBuildRetry                   don't retry any index builds that were 
                                        interrupted by shutdown
  --noscripting                         disable scripting engine
  --notablescan                         do not allow table scans
  --shutdown                            kill a running server (for init 
                                        scripts)

Replication options:
  --oplogSize arg                       size to use (in MB) for replication op 
                                        log. default is 5% of disk space (i.e. 
                                        large is good)

Master/slave options (old; use replica sets instead):
  --master                              master mode
  --slave                               slave mode
  --source arg                          when slave: specify master as 
                                        <server:port>
  --only arg                            when slave: specify a single database 
                                        to replicate
  --slavedelay arg                      specify delay (in seconds) to be used 
                                        when applying master ops to slave
  --autoresync                          automatically resync if slave data is 
                                        stale

Replica set options:
  --replSet arg                         arg is <setname>[/<optionalseedhostlist
                                        >]
  --replIndexPrefetch arg               specify index prefetching behavior (if 
                                        secondary) [none|_id_only|all]
  --enableMajorityReadConcern           enables majority readConcern

Sharding options:
  --configsvr                           declare this is a config db of a 
                                        cluster; default port 27019; default 
                                        dir /data/configdb
  --configsvrMode arg                   Controls what config server protocol is
                                        in use. When set to "sccc" keeps server
                                        in legacy SyncClusterConnection mode 
                                        even when the service is running as a 
                                        replSet
  --shardsvr                            declare this is a shard db of a 
                                        cluster; default port 27018

SSL options:
  --sslOnNormalPorts                    use ssl on configured ports
  --sslMode arg                         set the SSL operation mode 
                                        (disabled|allowSSL|preferSSL|requireSSL
                                        )
  --sslPEMKeyFile arg                   PEM file for ssl
  --sslPEMKeyPassword arg               PEM file password
  --sslClusterFile arg                  Key file for internal SSL 
                                        authentication
  --sslClusterPassword arg              Internal authentication key file 
                                        password
  --sslCAFile arg                       Certificate Authority file for SSL
  --sslCRLFile arg                      Certificate Revocation List file for 
                                        SSL
  --sslDisabledProtocols arg            Comma separated list of TLS protocols 
                                        to disable [TLS1_0,TLS1_1,TLS1_2]
  --sslWeakCertificateValidation        allow client to connect without 
                                        presenting a certificate
  --sslAllowConnectionsWithoutCertificates 
                                        allow client to connect without 
                                        presenting a certificate
  --sslAllowInvalidHostnames            Allow server certificates to provide 
                                        non-matching hostnames
  --sslAllowInvalidCertificates         allow connections to servers with 
                                        invalid certificates
  --sslFIPSMode                         activate FIPS 140-2 mode at startup

Storage options:
  --storageEngine arg                   what storage engine to use - defaults 
                                        to wiredTiger if no data files present
  --dbpath arg                          directory for datafiles - defaults to 
                                        /data/db
  --directoryperdb                      each database will be stored in a 
                                        separate directory
  --noprealloc                          disable data file preallocation - will 
                                        often hurt performance
  --nssize arg (=16)                    .ns file size (in MB) for new databases
  --quota                               limits each database to a certain 
                                        number of files (8 default)
  --quotaFiles arg                      number of files allowed per db, implies
                                        --quota
  --smallfiles                          use a smaller default file size
  --syncdelay arg (=60)                 seconds between disk syncs (0=never, 
                                        but not recommended)
  --upgrade                             upgrade db if needed
  --repair                              run repair on all dbs
  --repairpath arg                      root directory for repair files - 
                                        defaults to dbpath
  --journal                             enable journaling
  --nojournal                           disable journaling (journaling is on by
                                        default for 64 bit)
  --journalOptions arg                  journal diagnostic options
  --journalCommitInterval arg           how often to group/batch commit (ms)

WiredTiger options:
  --wiredTigerCacheSizeGB arg           maximum amount of memory to allocate 
                                        for cache; defaults to 1/2 of physical 
                                        RAM
  --wiredTigerStatisticsLogDelaySecs arg (=0)
                                        seconds to wait between each write to a
                                        statistics file in the dbpath; 0 means 
                                        do not log statistics
  --wiredTigerJournalCompressor arg (=snappy)
                                        use a compressor for log records 
                                        [none|snappy|zlib]
  --wiredTigerDirectoryForIndexes       Put indexes and data in different 
                                        directories
  --wiredTigerCollectionBlockCompressor arg (=snappy)
                                        block compression algorithm for 
                                        collection data [none|snappy|zlib]
  --wiredTigerIndexPrefixCompression arg (=1)
                                        use prefix compression on row-store 
                                        leaf pages
```
## 创建服务文件
* 在mongodb/bin目录下创建配置文件mongodb.conf
``` bash
[root@snails ~]# cd mongodb/bin
[root@snails bin]# vi mongodb.conf
#数据文件存放目录
dbpath = /data/mongodata
#日志文件存放目录
logpath = /data/log/mongolog/mongodb.log
#端口
port = 27017
#以守护程序的方式启用，即在后台运行
fork = true   
nohttpinterface = true
```
### 启动服务
``` bash
[root@snails bin]# mongod --dbpath=/data/mongodata --logpath=/data/log/mongolog/mongodb.log --logappend --fork
#通过配置文件启动
[root@snails bin]# mongod -f /root/mongodb/bin/mongodb.conf
netstat -tnlp | grep mongod
tcp        0      0 0.0.0.0:27017           0.0.0.0:*               LISTEN      18093/mongod 
```
### 测试
``` bash
[root@snails bin]# mongo
MongoDB shell version: 3.2.7
connecting to: test
Server has startup warnings: 
2016-07-07T20:38:09.623+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] 
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] 
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] 
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] 
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. rlimits set to 15084 processes, 65535 files. Number of processes should be at least 32767.5 : 0.5 times number of files.
2016-07-07T20:38:09.624+0800 I CONTROL  [initandlisten] 
> show dbs
local  0.000GB
> quit()
```
## 消除警告
``` bash
[root@snails bin]# vi /etc/rc.local
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
ulimit -u 65535
[root@snails bin]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@snails bin]# echo never > /sys/kernel/mm/transparent_hugepage/defrag
```
## 文件限制数调整
修改配置文件 /etc/security/limits.conf，添加配置信息：
``` bash
* soft nofile 65535
* hard nofile 65535
* soft nproc 32000
* hard nproc 32000
```
## 停止mongodb
``` bash
正常停止方法: kill  -2 PID 
>use  admin  
>db.shutdownServer();
```

## 再次验证
``` bash
[root@snails bin]# mongod -f /root/mongodb/bin/mongodb.conf
about to fork child process, waiting until server is ready for connections.
forked process: 18229
child process started successfully, parent exiting
[root@snails bin]# mongo
MongoDB shell version: 3.2.7
connecting to: test
Server has startup warnings: 
2016-07-07T21:06:53.798+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-07-07T21:06:53.798+0800 I CONTROL  [initandlisten] 
> exit
bye
```
