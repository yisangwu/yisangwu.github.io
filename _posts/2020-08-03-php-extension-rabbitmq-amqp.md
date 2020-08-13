---
layout: post
title: PHP安装RabbitMQ扩展amqp=1.10.2
tags: php, rabibtmq
categories: php
---

CentOS版本：
```shell
# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.6 (Maipo)
```
PHP版本：
```shell
# php -v
PHP 7.2.19 (cli) (built: Jun  4 2019 17:46:23) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```
pecl 安装失败，判断是没有预先安装amqp的依赖包rabbitmq-c
```shell
# pecl install amqp
```

# 1. 安装依赖 rabbitmq-c
* 1. 1 下载rabbitmq-c源码：
```shell
# wget https://github.com/alanxz/rabbitmq-c/archive/v0.10.0.tar.gz
```
* 1.2 解压：
```shell
# tar -zxf v0.10.0.tar.gz
```
* 1.3 进入源码目录：
```shell
# cd rabbitmq-c-0.10.0/
```
* 1.4 编译源码，指定目录：
```shell
# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/rabbitmq-c-0.10.0
...
-- Configuring done
-- Generating done
-- Build files have been written to: /root/rabbitmq/rabbitmq-c-0.10.0
```
编译安装：
```shell
# make && make install
...
[100%] Built target test_tables
Install the project...
-- Install configuration: "Release"
-- Installing: /usr/local/rabbitmq-c-0.10.0/lib64/pkgconfig/librabbitmq.pc
-- Installing: /usr/local/rabbitmq-c-0.10.0/lib64/librabbitmq.so.4.4.0
-- Installing: /usr/local/rabbitmq-c-0.10.0/lib64/librabbitmq.so.4
-- Installing: /usr/local/rabbitmq-c-0.10.0/lib64/librabbitmq.so
-- Installing: /usr/local/rabbitmq-c-0.10.0/lib64/librabbitmq.a
-- Installing: /usr/local/rabbitmq-c-0.10.0/include/amqp.h
-- Installing: /usr/local/rabbitmq-c-0.10.0/include/amqp_framing.h
-- Installing: /usr/local/rabbitmq-c-0.10.0/include/amqp_tcp_socket.h
```

# 2.  安装amqp

PECL中的amqp地址：http://pecl.php.net/package/amqp
> 最新版本：http://pecl.php.net/package/amqp/1.10.2 

* 2.1 下载amqp源码：
```shell
# wget http://pecl.php.net/get/amqp-1.10.2.tgz
```
* 2.2 解压amqp：
```shell
# tar -zxf amqp-1.10.2.tgz
```
* 2.3 进入源码目录：
```shell
# cd /root/amqp-1.10.2
```
* 2.4 找到phpize
```shell
# whereis phpize
phpize: /usr/bin/phpize
# ll /usr/bin/phpize 
lrwxrwxrwx. 1 root root 25 Jun  4  2019 /usr/bin/phpize -> /usr/local/php/bin/phpize
```
* 2.5 执行phpize 命令是，准备 PHP 扩展库的编译环境：
```shell
# /usr/local/php/bin/phpize
```
* 2.6 安装扩展：
```shell
# ./configure  --with-php-config=/usr/local/php/bin/php-config
checking for pkg-config... /usr/bin/pkg-config
checking for amqp using pkg-config... configure: error: librabbitmq not found
```

* 2.7 继续编译安装amqp：
```shell
# cd /root/amqp-1.10.2
# ./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.10.0
# make && make install
...
/usr/bin/ld: cannot find -lrabbitmq
collect2: error: ld returned 1 exit status
make: *** [amqp.la] Error 1
```
* 2.8 查看rabbitmq-c目录：
```shell
# ll /usr/local/rabbitmq-c-0.10.0
total 0
drwxr-xr-x 2 root root  92 Aug  3 10:28 include
drwxr-xr-x 3 root root 118 Aug  3 10:28 lib64
```
* 2.9 复制一份lib64 为lib：
```shell
# cp -R /usr/local/rabbitmq-c-0.10.0/lib64/ /usr/local/rabbitmq-c-0.10.0/lib/
# ll /usr/local/rabbitmq-c-0.10.0/
total 0
drwxr-xr-x 2 root root  92 Aug  3 10:28 include
drwxr-xr-x 3 root root 118 Aug  3 10:38 lib
drwxr-xr-x 3 root root 118 Aug  3 10:28 lib64
```
* 2.10 继续编译安装amqp：
```shell
# make && make install
...
Build complete.
Don't forget to run 'make test'.

Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20170718/
```
* 2.11 查看扩展文件：
```shell
# ll /usr/local/php/lib/php/extensions/no-debug-non-zts-20170718/
...
extension=amqp.so 
...
```
* 2.12 添加扩展文件到php.ini文件中：extension=amqp.so  
a.  直接在php.ini中添加：
找到php使用的配置文件：
```shell
# php --ini
Configuration File (php.ini) Path: /usr/local/php/etc
Loaded Configuration File:         /usr/local/php/etc/php.ini
Scan for additional .ini files in: /usr/local/php/conf.d
Additional .ini files parsed:      /usr/local/php/conf.d/007-redis.ini,
/usr/local/php/conf.d/amqp.ini,
/usr/local/php/conf.d/mongodb.ini
```  
  b.  php.ini 的搜索路径中，独立添加：
  ```shell
# php -i | grep configure
--with-config-file-path=/usr/local/php/etc
```
 - 在 /usr/local/php/etc 中新建一个文件，添加扩展：
```shell
# cat /usr/local/php/conf.d/amqp.ini 
extension = "amqp.so"
```
* 2.13 重载php配置(平滑重启)：
```shell
# service php-fpm reload
Reload service php-fpm  done
```
* 2.14 查看php扩展中是否已存在amqp：
```shell
# php -m | grep amqp
amqp
...
```
* 2.15 查看amqp扩展信息
```shell
# php --ri amqp
amqp
Version => 1.10.2
Revision => release
...
```
