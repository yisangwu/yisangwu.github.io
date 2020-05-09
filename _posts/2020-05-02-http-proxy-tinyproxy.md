---
layout: post
title: web 代理 Tinyproxy
tags: linux centos tinyproxy
categories: centos 
---

环境：centOS7.7

为业务需要，部署了一个轻量级的HTTP(s)代理，作为业务服务器请求第三方服务的代理。

1. 查看当前yum源中有无tinyproxy：
```shell
# yum list | grep tinyproxy
```
2. 下载centos7的 epel软件包：
```shell
# wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
3 安装rpm软件包：
```shell
# rpm -iUvh epel-release-latest-7.noarch.rpm
```
4. 查看tinyproxy版本：
```shell
# yum list | grep tinyproxy
tinyproxy.x86_64    1.8.3-2.el7    epel
```
5. yum安装：
```shell
# yum install  -y tinyproxy
```
6. 修改配置：

tinyproxy 默认的配置文件路径：
```shell
 /etc/tinyproxy/tinyproxy.conf
 ```

根据情况，调整的关键几个配置项：
```shell
Port 8888    # 监听的端口，默认为8888，如果要支持公网访问，则需要开放云服务器端口的公网访问   
Allow 127.0.0.1     # 只允许本机使用，如有多个可写多个Allow。注释掉，则允许所有ip连接。  
Listen 192.168.0.1  # 监听的IP， 一般为本机内网IP  
MaxClients 100      # 同一时间的最大客户端连接数  
MinSpareServers 5   # 最小空闲进程数  
MaxSpareServers 20  # 最大空闲进程数  
StartServers 10     # 初始化启动的进程数  
MaxRequestsPerChild 0   # 单个进程处理的最大连接数， 0 为无限制
```

7. 启动tinyproxy：
```shell
$ systemctl  start tinyproxy 
```
8. 设置开机自启
```shell
$ systemctl enable tinyproxy
```
9. 查看端口监听， 8888 为tinyproxy： 
```shell
# netstat -lnpt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      961/sshd  
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      5313/tinyproxy   
```

附管理命令：
* 重启
```shell
$ systemctl restart tinyproxy
```
* 停止：
```shell
$ systemctl stop tinyproxy
```
* 查看状态：
```shell
$ systemctl status tinyproxy
```

------------
### 问题:
代理不可用，业务访问超时无响应,  重启 tinyproxy问题解决。  
查看日志，有如下记录：
```shell
NOTICE    May 01 19:08:33 [17728]: Waiting servers (0) is less than MinSpareServers (5). Creating new child.
NOTICE    May 01 19:08:38 [17728]: Waiting servers (0) is less than MinSpareServers (5). Creating new child.
NOTICE    May 01 19:08:43 [17728]: Waiting servers (0) is less than MinSpareServers (5). Creating new child.
```
意思是当前服务器无空闲进程可用。查看业务访问数据，实际上却没有太多的代理访问。
安装tinyproxy时没有调整日志级别，用的是默认的：
```shell
LogLevel Info
```
观察日志，发现服务器上正常的http访问也占用了代理，调整 Listen监听的ip为本机内网IP，重启tinyproxy后，问题解决。

> 附：google了下，有解决方案是起个crontab，每小时重启下tinyproxy，防止tinyproxy死掉。  

如果在重启周期内死掉了，在重启恢复前的那段时间，代理不可用，业务还是会受到影响的。