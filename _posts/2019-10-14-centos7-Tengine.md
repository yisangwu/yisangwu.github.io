
---
layout: post
title: CentOS7 安装淘宝 Tengine，健康检查负载均衡
tags: linux, centos, Tengine, nginx
categories: centos
---


Tcp代理加负载均衡走了Haproxy后，  
Http的代理和负载均衡，想换个方式，nginx也可以做，但是健康检查太被动了，失败了才发现异常，然后再切换。并不是真正意义上的健康检查。

所以准备使用Tengine来替代nginx，来做http七层转发。  
nginx 并没有自带 ”健康检查“， 而是在某节点出现异常时进行切换。
Tengine的health_check_module 则是主动发起请求，去探测是否符合配置预期。   
ngx_http_upstream_check_module此功能为tengine独立实现的主动健康
检查功能，Tengine和nginx默认的被动健康检查不同。此功能nginx plus商业版也具备，但是闭源收费的。

## Tengine
> 官方简介： Tengine是由淘宝网发起的Web服务器项目。它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如淘宝网，天猫商城等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。

官网及下载地址：[https://tengine.taobao.org/](https://tengine.taobao.org/)

服务器环境：
> CentOS Linux release 7.8

1.  下载Tengine源码：
```shell
$ wget https://tengine.taobao.org/download/tengine-2.3.2.tar.gz
```
2.  解压：
```shell
$ tar -zxf tengine-2.3.0.tar.gz
```
3.  进入源码目录：
```shell
$ cd tengine-2.3.2
```
4.  编译配置文件，添加监控检查模块，安装目录使用默认的/usr/local/nginx：
```shell
$ ./configure --add-module=./modules/ngx_http_upstream_check_module
# 如果要修改安装目录，添加参数 --prefix=指定目录的绝对路径
```

### 注意
> 官方文档写的是： 该模块可以为Tengine提供主动式后端服务器健康检查的功能。 该模块在Tengine-1.4.0版本以前没有默认开启，它可以在配置编译选项的时候开启：./configure --with-http\_upstream\_check_module
> 
> > 执行时，会报错： $ ./configure --with-http\_upstream\_check\_module   
./configure: error: invalid option "--with-http\_upstream\_check\_module"

5.  编译安装：
```shell
$ make && make install
```

6.  查看安装版本：

```shell
# nginx -V
Tengine version: Tengine/2.3.2
nginx version: nginx/1.17.3
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --add-module=./modules/ngx_http_upstream_check_module
```

### 常用命令：

-   显示所有支持的指令:

```shell
$ nginx -l
```

-   显示所有加载的模块：

```shell
$ nginx -m
```

-   检查nginx的配置文件是否正确

```shell
$ nginx -t
```

-   启动nginx
```shell
$ nginx -s
```

-   停止nginx

```shell
$ nginx -s stop
```

-   重新加载配置文件

```shell
$ nginx -s reload
```

> 附： 
>> 健康检查的 Tengine配置，后续再写
