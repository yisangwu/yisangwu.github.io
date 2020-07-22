---
layout: post
title: mysqldump使用rsync异地全量备份数据库
tags: linux CentOS mysql
categories: CentOS
---
Mysql 做了本机，同机房全量备份后，想同时做一份异地备份,考虑异地机房的网络不稳定性，使用 rsync 来同步备份的文件；  
同机房异机备份，内网速度快可以使用scp，或者mysqldump备份脚本放在非mysql机器上；

> 服务器环境： CentOS Linux release 7.7.1908 (Core)  
> 当前是一体机，所以在mysql机器做mysqldump，实际业务中上最好在非mysql机器执行mysqldump做备份；

### 一. 远程备份服务器，安装rsync：

1.1 使用yum安装：
```shell
[root]# yum install -y rsync
```
1.2 查看版本：
```shell
[root]# rsync --version
rsync  version 3.1.3  protocol version 31
```
1.3 查询配置文件存放的路径
```shell
[root]# rpm -qc rsync
/etc/rsyncd.conf
/etc/sysconfig/rsyncd
```
1.4 查看systemd服务管理脚本：
```shell
[root]# ll /lib/systemd/system | grep rsync
-rw-r--r--  1 root root  237 Apr  1 12:22 rsyncd.service
-rw-r--r--  1 root root  220 Apr  1 12:22 rsyncd@.service
-rw-r--r--  1 root root  138 Apr  1 12:22 rsyncd.socket
```
> 在/lib/systemd/system目录下存在rsyncd.service， 即可以使用systemctl 管理 rsync

1.5 开放服务器公网访问端口，默认使用873，如果修改rsync端口，则开放相应端口;
> >重中之重

### 二. 远程备份服务器，配置rsync：
2.1 修改rsyncd.conf配置：
```shell
[root]# cat /etc/rsyncd.conf
# /etc/rsyncd: configuration file for rsync daemon mode
# See rsyncd.conf man page for more options
# configuration example:
# port=873 默认端口，修改端口，需要开放公网访问
uid = root
gid = root
use chroot = no
max connections = 4  
timeout = 900  # 超时时间
ignore nonreadable = yes
log file=/var/log/rsyncd.log  # log存放路径
pid file=/var/run/rsyncd.pid  # pid存放路径
lock file=/var/run/rsyncd.lock  # 锁文件存放路径
secrets file=/etc/rsyncd.passwd  # 密码文件

[mysql]  # 模块名
comment=MySQL remote backup  # 模块注释
path = /data-backup/mysql  # 绝对路径
read only=no   # 默认为yes，客户端不能上传，只能下载
auth users=user_rsync  # 认证用户

```
2.2 创建密码文件，rsync认证的用户和密码, 不需要使用useradd真实创建用户：
```shell
[root]#echo 'user_rsync:Rsync@123'>/etc/rsyncd.passwd
```
2.3 修改认证为root的读写权限：
```shell
[root]#chmod 600 /etc/rsyncd.passwd
```

### 三. 远程备份服务器，启动rsync：
3.1 使用systemctl管理服务：
```shell
[root]# systemctl start rsyncd
```
> 重启：# systemctl restart rsyncd
> 查看状态：# systemctl status rsyncd

3.2 查看端口监听状态：
```shell
[root]# netstat -lnpt | grep 873
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN      2392/rsync          
tcp6       0      0 :::873                  :::*                    LISTEN      2392/rsync 
```
3.3 查看rsync的log，是否正常启动：
```shell
[root]# cat /var/log/rsyncd.log 
2020/04/01 12:29:46 [8159] rsyncd version 3.1.3 starting, listening on port 873
```
### 四. Mysql服务器，创建备份用户：

3.1 使用mysql的root账号，进入终端：
```shell
[root]#mysql -u root -p
```
3.2 创建mysqldump用户，建议和业务数据库用户区分开, GRANT权限可以调整，注意一定不要 GRANT ALL:
```mysql
mysql> use mysql -A;
mysql> CREATE USER 'mysql_backup'@'localhost' identified by "Backup@Abc123";
mysql> GRANT SELECT, RELOAD, REPLICATION CLIENT, SHOW VIEW, TRIGGER ON *.* TO 'mysql_backup'@'localhost';
mysql> FLUSH PRIVILEGES;
```
### 五. Mysql服务器，创建备份脚本：
5.1 创建备份脚本，可以根据业务实际情况，修改：

``` shell
[root opt]# vim /opt/mysql_backup.sh
#!/bin/bash

# mysql config
db_host='localhost'
db_user='mysql_backup'
db_password='Backup@Abc123'

# backup file
bk_time=`date +%Y%m%d_%H%M%S`
bk_file="/opt/mysql-bk-app-${bk_time}.sql.gz"

# delete backup 7 days before
find /opt/ -name "*.sql.gz" -type f -mtime +7 -exec rm -rf {} \; > /dev/null 2>&1

# backup all database
mysqldump --opt -h${db_host} -u${db_user} -p${db_password} --default-character-set=utf8mb4 --all-databases --single-transaction --flush-logs --master-data=2 | gzip >${bk_file}

# rsync to remote machine
nohup rsync -az --include="*.gz" --exclude=* --password-file=/etc/rsyncd.passwd /opt/ user_rsync@1.2.3.4::mysql >/dev/null 2>&1 &
```
> 同机房异机备份，内网速度快可以使用scp，或者备份脚本放在非mysql机器上；  
> a. 如果mysql没有开启binlog，需去掉binlog刷新：--flush-logs --master-data=2  
> b. user_rsync(at)10.20.30.40::mysql ( 认证用户@远程服务器::模块名 )  
> c. --include="*.gz" 只同步gz后缀得文件，脚本等其他文件不同步  
> d. --exclude=* 排除目录下所有文件， 注意exclude 必须在 include后面

5.2 修改/opt/mysql_backup.sh脚本可执行：
```shell
[root]#chmod +x /opt/mysql_backup.sh
```

### 六. Mysql服务器，测试脚本，添加crontab：

6.1 测试脚本，查看远程备份服务器的文件：
```shell
[root]#/opt/mysql_backup.sh
```
> 在远程服务器，mysql模块对应的目录，rsync文件传输中是隐藏文件，要使用 ll -la 查看。

6.2 添加定时脚本：
```shell
[root]# crontab -e
#mysql remote backup
00 3 * * * /opt/mysql_backup.sh > /opt/x_mysql_backup.log 2>&1
```
> 每天03:00，执行一次, 具体根据业务情况，调整脚本执行时间和频率。
