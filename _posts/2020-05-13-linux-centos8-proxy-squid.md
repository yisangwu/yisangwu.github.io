---
layout: post
title: CentOS8安装Squid代理服务器
tags: centos, squid
categories: centos
---

更换了一台阿里云服务器，重新部署服务。直接上了CentOS8系统。

CentOS8系统除了默认的pyhton版本是 Python 3.6 ，RHEL 包含PHP 7.2。

结果，原来的python2的服务都要重新兼容一次python3，也懒得再去安装pyhton2版本。

更重要的是，原本代理服务Tinyproxy，在CentOS8上面，无论是rpm安装，还是源码安装最新版本Version 1.10.0，都起不来！！！

> Tinyproxy git地址：https://github.com/tinyproxy  
> rpm支持EPEL 7 for x86_64的版本：tinyproxy-1.8.3-2.el7.x86_64.rpm

不是 PID 未创建，就是 protocol错误。

本想开--enable-debug调试模式研究下， 看下怎么解决。但是业务优先，立刻更换代理，换上 squid。

squid
Squid是一个高性能的代理缓存服务器，Squid支持FTP、gopher、HTTPS和HTTP协议。和一般的代理缓存软件不同，Squid用一个单独的、非模块化的、I/O驱动的进程来处理所有的客户端请求。

squid官网：http://www.squid-cache.org/

squid下载地址：http://www.squid-cache.org/Versions/

squid的rpm文件：http://rpmfind.net/linux/rpm2html/search.php?query=squid



服务器环境：
```shell
$ cat /etc/redhat-release 
CentOS Linux release 8.1.1911 (Core)
```
yum源中的版本：
```shell
$ yum list | grep squid
squid.x86_64              7:4.4-8.module_el8.1.0+197+0c39cdc8               @AppStream 
squidGuard.x86_64         1.4-36.el8                                        epel 
```
直接yum安装：
```shell
$ yum install -y squid
```
查看已安装：
```shell
$ whereis squid
```
查看启动脚本：
```shell
$ ll /usr/lib/systemd/system | grep squid
```
进入配置文件目录：
```shell
$ cd /etc/squid/
```
编辑配置文件，允许所有ip访问：
```shell
$ vim squid.conf 
http_port 3228 # 修改端口，默认为 3128 
http_access allow all # 添加 ，为允许所有ip
```
检查配置文件是否有误：
```shell
$ squid -k parse  #　配置文件解析日志中，没有出现ERROR 就没有问题
```
启动服务：
```shell
$ systemctl start squid
```shell
查看squid监听的端口：
```shell
$ netstat -lnpt
```
squid的日志目录为/var/log/squid/， 两种类型日志access 和cache；
使用代理 http 或 https：
```shell
公网IP：监听端口  
```
管理命令：
```shell
$ squid -k parse # 检查配置文件是否有误
$ systemctl start squid # 启动 squid
$ systemctl status squid # 查看 squid 运行状态
$ systemctl stop squid # 停止 squid
$ systemctl restart squid # 重启 squid
```
---

进阶配置，只允许指定IP使用squid：

1. 创建ip白名单文件，/etc/squid/squid_allow_ips， 每个ip一行，注意文件的用户和组权限（ squid.root）：
```shell
$ cat /etc/squid/squid_allow_ips
1.2.3.4
2.2.3.4
3.2.3.4
```
2. 修改squid配置，添加acl 访问规则：
```shell
acl allowed_ips src "/etc/squid/squid_allow_ips"  # ip白名单
http_access allow localnet
http_access allow localhost
http_access allow allowed_ips  # 配置allow
# And finally deny all other access to this proxy
http_access deny all  # 禁止所有访问，这个不要漏了
```
3. 检查配置文件是否有误：
```shell
$ squid -k parse  #　配置文件解析日志中，没有出现ERROR 就没有问题
```
4. 重新启动服务：
```shell
$ systemctl restart squid
```