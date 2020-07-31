---
layout: post
title: centOS7安装rabbitmq3.8.5
tags: centos, rabbitmq
categories: centos
---

## CentOS环境：
1. redhat的release版本:
```shell
[root@depakin ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
```
2. Linux内核版本：
```shell
[root@depakin ~]# uname -a
Linux fuckqiu 3.10.0-1127.10.1.el7.x86_64 #1 SMP Wed Jun 3 14:28:03 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

## RabbitMQ相关地址：    
RabbiMQ官网下载及安装说明地址：https://www.rabbitmq.com/download.html  
> 依赖：erlang， socat，logrotate（系统默认）  
> erlang版本依赖说明：https://www.rabbitmq.com/which-erlang.html  
> 3.8.5  需要的erlang版本号，最小21.3，最大 23.x  

RabbitMQ下载：
- github下载，：  
https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.8.5 
> github里面有rpm，exe，源码
- package cloud中rabbitmq仓库地址：  
https://packagecloud.io/rabbitmq 
> packagecloud.io里面有rabbitmq-server， erlang等仓库和个版本的rpm

---

## 1. 安装依赖：
```shell
# yum install -y socat
```

## 2. 安装Erlang：

1. 下载地址：
* github地址：https://github.com/rabbitmq/erlang-rpm/tree/v23.0.3
* package cloud地址：https://packagecloud.io/rabbitmq/erlang

2. 下载erlang 23.0.3：
```shell
# wget --content-disposition https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-23.0.3-1.el7.x86_64.rpm/download.rpm
```
3. 安装erlang:
```shell
# rpm -ivh erlang-23.0.3-1.el7.x86_64.rpm
```
4. 查找erlang:
```shell
# whereis erlang
erlang: /usr/lib64/erlang   
```
5. 查询已安装erlang包信息：
```shell
# rpm -qa erlang
erlang-23.0.3-1.el7.x86_64
```
6. 启动 erlang 的 shell：
```shell
[root@fuckqiu ~]# erl
Erlang/OTP 23 [erts-11.0.3] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1] [hipe]
Eshell V11.0.3  (abort with ^G)
1> 1+1.
2
2> halt().  # 关闭 Erlang 系统
```

## 3. 安装RabbitMQ：

1. 下载3.8.5版本的rpm， noarch为适用所有CPU：
```shell
# wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.5/rabbitmq-server-3.8.5-1.el7.noarch.rpm
```
2. github.com访问失败，则尝试使用：
```shell
# wget --content-disposition https://packagecloud.io/rabbitmq/rabbitmq-server/packages/el/7/rabbitmq-server-3.8.5-1.el7.noarch.rpm/download.rpm
```

3. 安装rabbitMQ:
```shell
# rpm -ivh ./rabbitmq-server-3.8.5-1.el7.noarch.rpm
warning: ./rabbitmq-server-3.8.5-1.el7.noarch.rpm: Header V4 RSA/SHA256 Signature, key ID 6026dfca: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:rabbitmq-server-3.8.5-1.el7      ################################# [100%]
```
4. 查找rabbitmq：
```shell
# whereis rabbitmq
rabbitmq: /usr/lib/rabbitmq /etc/rabbitmq
```
5. 列出rabbitmq执行文件：
```shell
# ll /usr/sbin/ | grep 'rabbit'
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmqctl
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmq-diagnostics
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmq-plugins
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmq-queues
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmq-server
-rwxr-xr-x  1 root root      3106 Jun 15 21:42 rabbitmq-upgrade
```

