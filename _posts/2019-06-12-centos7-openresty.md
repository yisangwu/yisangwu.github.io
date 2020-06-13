---
layout: post
title: CentOS7安装OpenResty
tags: centos, openresty
categories: centos
---
<style type="text/css">
    p{text-indent: 20px}
</style>
> 环境：CentOS Linux release 7.5

有一个项目是Lua库写的，需要部署一个web服务器，考虑Openresty对lua的支持，
就部署Openresty。

### OpenResty 
<p>官方说明：OpenResty® 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。</p>

### 下载地址：
- OpenResty官网： <http://openresty.org/>
- OpenResty下载地址：
<http://openresty.org/cn/download.html>

1. 安装依赖库：
```shell
$ yum install -y pcre-devel openssl-devel gcc curl
```
2. 下载版本：
```shell
$ wget https://openresty.org/download/openresty-1.15.8.1.tar.gz
```
3. 解压：
```shell
$ tar -xzvf openresty-1.15.8.1.tar.gz
```
4. 进入解压目录：
```shell
$ cd openresty-1.15.8.1/
```
5. 检查配置环境, 生成 Makefile，默认安装到/usr/local/openresty：
```shell
$ ./configure
```
6. 编译安装：
```shell
$ gmake && gmake install
```
安装结果：
```shell
mkdir -p /usr/local/openresty/site/lualib /usr/local/openresty/site/pod /usr/local/openresty/site/manifest
ln -sf /usr/local/openresty/nginx/sbin/nginx /usr/local/openresty/bin/openresty
```
可以看到openresty 实际上是nginx的软连接。

7. 查看版本号：
```shell
$ /usr/local/openresty/bin/openresty -v
nginx version: openresty/1.15.8.3
```

8. 启动：
```shell
检验配置
/usr/local/openresty/bin/openresty -t
启动
/usr/local/openresty/bin/openresty
停止
/usr/local/openresty/bin/openresty -s stop
重新加载配置
/usr/local/openresty/bin/openresty -s reload
```

9. 配置/lib/systemd/system/openresty.service，通过systemctl启动：

```shell
[Unit]
Description=openresty - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/openresty/bin/openresty -c /usr/local/openresty/nginx/conf/nginx.conf
ExecReload=/usr/local/openresty/bin/openresty -s reload
ExecStop=/usr/local/openresty/bin/openresty -s stop

[Install]
WantedBy=multi-user.target
```

添加openresty.service后，是配置生效：
```shell
# systemctl daemon-reload
```

然后，就可以使用systemctl管理openresty:

```shell
启动
systemctl start openresty
停止
systemctl stop openresty
重载配置
systemctl reload openresty
重启
systemctl restart openresty
```
