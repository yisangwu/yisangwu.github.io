---
layout: post
title: CentOS7安装RabbitMQ的PHP扩展amqp==1.10.2
tags: centos,linux,php, rabbitmq, amqp
categories: php
---


### CentOS版本：
```shell
# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.6 (Maipo)
```
### PHP版本：
```shell
# php -v
PHP 7.2.19 (cli) (built: Jun  4 2020 17:46:23) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```
pecl 安装失败，判断是没有预先安装amqp的依赖包rabbitmq-c:
```shell
# pecl install amqp
```
PECL中的amqp地址：
http://pecl.php.net/package/amqp
> 最新版本：http://pecl.php.net/package/amqp/1.10.2

## amqp：

下载amqp源码：
```shell
# wget http://pecl.php.net/get/amqp-1.10.2.tgz
```
解压amqp：
```shell
# tar -zxf amqp-1.10.2.tgz
```
进入amqp源码目录：
```shell
# cd /root/amqp-1.10.2
```
找到phpize：
```shell
# whereis phpize
phpize: /usr/bin/phpize
# ll /usr/bin/phpize 
lrwxrwxrwx. 1 root root 25 Jun  4  2020  /usr/bin/phpize -> /usr/local/php/bin/phpize
```
在amqp源码目录，执行phpize 命令是，准备 PHP 扩展库的编译环境：
```shell
# /usr/local/php/bin/phpize
```
安装amqp扩展：
```shell
# ./configure  --with-php-config=/usr/local/php/bin/php-config
checking for pkg-config... /usr/bin/pkg-config
checking for amqp using pkg-config... configure: error: librabbitmq not found
```
报错没有找到librabbitmq，需要安装rabbitmq-c。

## 安装rabbitmq-c：

下载rabbitmq-c源码：
```shell
# wget https://github.com/alanxz/rabbitmq-c/archive/v0.10.0.tar.gz
```
解压rabbitmq-c：
```shell
# tar -zxf v0.10.0.tar.gz
```
进入rabbitmq-c源码目录：
```shell
# cd rabbitmq-c-0.10.0/
```
编译rabbitmq-c源码，指定目录：
```shell
# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/rabbitmq-c-0.10.0
...
-- Configuring done
-- Generating done
-- Build files have been written to: /root/rabbitmq/rabbitmq-c-0.10.0
```
编译安装rabbitmq-c：
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

继续编译安装amqp：
```shell
# cd /root/amqp-1.10.2
# ./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.10.0
# make && make install
...
/usr/bin/ld: cannot find -lrabbitmq
collect2: error: ld returned 1 exit status
make: *** [amqp.la] Error 1
```
查看rabbitmq-c目录：
```shell
# ll /usr/local/rabbitmq-c-0.10.0
total 0
drwxr-xr-x 2 root root  92 Aug  3 10:28 include
drwxr-xr-x 3 root root 118 Aug  3 10:28 lib64
```
复制一份lib64 为lib：
```shell
# cp -R /usr/local/rabbitmq-c-0.10.0/lib64/ /usr/local/rabbitmq-c-0.10.0/lib/
# ll /usr/local/rabbitmq-c-0.10.0/
total 0
drwxr-xr-x 2 root root  92 Aug  3 10:28 include
drwxr-xr-x 3 root root 118 Aug  3 10:38 lib
drwxr-xr-x 3 root root 118 Aug  3 10:28 lib64
```
继续编译安装amqp：
```shell
#  cd /root/amqp-1.10.2
# make && make install
...
Build complete.
Don't forget to run 'make test'.

Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20170718/

查看扩展文件：
```shell
# ll /usr/local/php/lib/php/extensions/no-debug-non-zts-20170718/
...
extension=amqp.so 
...
```
## 添加amqp扩展文件到php.ini文件中：
> extension=amqp.so

### 1. 直接在php.ini中添加：
- 找到php使用的配置文件：
```shell
# php --ini
Configuration File (php.ini) Path: /usr/local/php/etc
Loaded Configuration File:         /usr/local/php/etc/php.ini
Scan for additional .ini files in: /usr/local/php/conf.d
```
- 添加  extension=amqp.so
```shell
# vim /usr/local/php/etc/php.ini
```
###  2. php.ini 的搜索路径中，独立添加：
- 找到php.ini 的搜索目录：
```shell
# php -i | grep configure
--with-config-file-path=/usr/local/php/etc
```
- 在 /usr/local/php/etc 中新建一个文件，添加扩展：
```shell
# cat /usr/local/php/conf.d/amqp.ini 
extension = "amqp.so"
```
- 重载php配置(平滑重启)：
```shell
# service php-fpm reload
Reload service php-fpm  done
```
- 查看php扩展中是否已存在amqp：
```shell
# php -m | grep amqp
amqp
...
```
-  查看amqp扩展信息
```shell
# php --ri amqp
amqp
Version => 1.10.2
Revision => release
...
```

