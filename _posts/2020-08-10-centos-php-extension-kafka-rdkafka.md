---
layout: post
title: CentOS下安装Kafka的PHP扩展rdkafka
tags: centos, php, rdkafka
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

kafka官方PHP客户端地址：
> https://cwiki.apache.org/confluence/display/KAFKA/Clients#Clients-PHP

# 1. librdkafka
> 安装kafka所有第三方客户端的依赖库：
> Kafka client based on librdkafka

#### 1.1 下载librdkafka源码：
```shell
# wget https://github.com/edenhill/librdkafka/archive/v1.5.0.tar.gz
#### 1.2  解压到当前文件夹：
```shell
# tar -zxf v1.5.0.tar.gz
```
#### 1.3  进入源码目录：
```shell
$ cd librdkafka-1.5.0/
```
#### 1.4 执行检测编译配置：
```shell
$ ./configure 
```
#### 1.5 编译：
```shell
# make
```
#### 1.6 安装：
```shell
# make install 
```
#### 1.7 查找已安装：
```shell
# whereis librdkafka
librdkafka: /usr/local/lib/librdkafka.a /usr/local/lib/librdkafka.so
```

# 2. php扩展 rdkafka

> pecl 地址： http://pecl.php.net/package/rdkafka


#### 2.1 使用pecl 安装 rdkafka 最新版本：
```shell
# pecl install rdkafka
Build process completed successfully
Installing '/usr/local/php/lib/php/extensions/no-debug-non-zts-20170718/rdkafka.so'
install ok: channel://pecl.php.net/rdkafka-4.0.3
Extension rdkafka enabled in php.ini
```
#### 2.2 查看php扩展是否安装：
```shell
# php -m | grep rdkafka
rdkafka
```
#### 2.3 查看当前php使用的配置：
```shell
# php --ini
Configuration File (php.ini) Path: /usr/local/php/etc
Loaded Configuration File:         /usr/local/php/etc/php.ini
Scan for additional .ini files in: /usr/local/php/conf.d
Additional .ini files parsed:      /usr/local/php/conf.d/007-redis.ini
```
#### 2.4 在php.ini中查找 rdkfka.so：
```shell
# grep 'rdkafka' /usr/local/php/etc/php.ini
extension="rdkafka.so"
```
#### 2.5 查看rdkafka扩展信息：
```shell
# php --ri rdkafka

rdkafka

rdkafka support => enabled
version => 4.0.3
build date => Aug 13 2020 10:29:54
librdkafka version (runtime) => 1.5.0
librdkafka version (build) => 1.5.0.255
```
