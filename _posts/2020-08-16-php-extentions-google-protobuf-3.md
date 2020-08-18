---
layout: post
title: PHP使用 Google Protocol Buffers (protobuf)
tags: php, protobuf
categories: php
---

很久之前，写PHP的时候，使用 Protobuf 做了聊天APP， 游戏服务器。 那个时候还用的是protobuf 2.5。  
看了下proto3的语法，来测试下：

服务器环境 与 protoc 版本：
```shell
# cat /etc/redhat-release 
    CentOS Linux release 7.8.2003 (Core)

# protoc --version
    libprotoc 3.11.4
```


## 一、安装 PHP 的  Protocol Buffers 扩展：

### 1.1 默认安装最新版本
```shell
# pecl install protobuf
```
### 1.2 指定版本号安装：
```shell
# pecl install protobuf-{VERSION}
```
### 1.3 查看扩展是否已安装：
```shell
# php -m | grep protobuf
protobuf
```
### 1.4 查看protobuf扩展的版本信息：
```shell
# php --ri protobuf

protobuf

Version => 3.13.0

Directive => Local Value => Master Value
protobuf.keep_descriptor_pool_after_request => 0 => 0
```

## 二、编写编译proto文件：

### 2.1 编写proto文件：
```shell
# cat pack.proto

syntax = "proto3";  // 语法协议
package pack.base;  // 命名空间

// 枚举，登录类型
enum LoginType{
    GUEST = 0;
    APPLE = 1;
    RENREN = 2;
    FACEBOOK = 3;
    KAIXIN = 4;
    GAMECENTER = 5;
    SINA = 6;
    WEICHAT = 7;
    ALIPAY = 8;
}

message LoginReq{
    string sig = 1;
    LoginType login_type = 2;
    string openid = 3;
    string channel = 4;
    string version = 5;
    string device_id = 6;
    string mac_id = 7;
    repeated int32 login_num = 8;
}
```
> 注意： proto3 仅仅支持 repeated字段修饰，如果使用required，optional编译会报错。

### 2.2 在proto文件目录，创建编译文件夹：
```shell
# mkdir proto_php
```

### 2.3 编译proto文件，生成 php类：
```shell
# protoc --php_out=./proto_php  pack.proto
```
>> 目录结构如下：
```shell
├─proto_php
│  ├─GPBMetadata
│  │   -- Pack.php
│  │
│  └─Pack
│      └─Base
│          -- LoginReq.php
│          -- LoginType.php
├─ test_proto.php
```

## 三、测试脚本：

```php
# cat test_proto.php
<?php

defined('DS') or define('DS', DIRECTORY_SEPARATOR);
/**
 * protobuf 测试, php
 * 
 * 编译proto文件，生成 php类
 * # protoc --php_out=./proto_php  pack.proto
 */

require __DIR__ . DS . 'proto_php' .DS. 'GPBMetadata' . DS . 'Pack.php';
require __DIR__ . DS . 'proto_php' .DS. 'Pack' . DS . 'Base' . DS .'LoginReq.php';
require __DIR__ . DS . 'proto_php' .DS. 'Pack' . DS . 'Base' . DS . 'LoginType.php';

use Pack\Base\{LoginReq, LoginType};

// 实例化 message类
$login_req = new LoginReq();

$login_req->setSig(md5(microtime(true)));
$login_req->setLoginType(LoginType::APPLE);
$login_req->setOpenid(sprintf('openid_%u', mt_rand(1000, 29520)));
$login_req->setChannel(sprintf('channel_00%u', mt_rand(1000, 29520)));
$login_req->setVersion(sprintf('V.%u.%u', mt_rand(1, 3), mt_rand(4, 20)));
$login_req->setDeviceId(sprintf('A00-B%s-C%s-D%s', mt_rand(1,6), mt_rand(8,70), mt_rand(100,237)));
$login_req->setMacId(str_pad('MAC', 20));
$login_req->setLoginNum(range(1,10));

// 序列化成二进制字符串
$data = $login_req->SerializeToString();

// 解析
$login_rsp = new LoginReq();
// 二进制字符串反序列化
$login_rsp->mergeFromString($data);

echo 'sig: ', $login_rsp->getSig(), PHP_EOL;
echo 'login_type: ', $login_rsp->getLoginType(), PHP_EOL;
echo 'openid: ', $login_rsp->getOpenid(), PHP_EOL;
echo 'channel: ', $login_rsp->getChannel(), PHP_EOL;
echo 'version: ', $login_rsp->getVersion(), PHP_EOL;
echo 'device_id: ', $login_rsp->getDeviceId(), PHP_EOL;

// repeated 类型的处理：
echo 'login_num size: ', count($login_rsp->getLoginNum()), PHP_EOL;

$login_num = [];
foreach ($login_rsp->getLoginNum() as $key => $value) {
    $login_num[] = $value;
}

echo 'login_num: ', implode(',', $login_num), PHP_EOL;
```

输入结果如下：
```shell
# php -f test_proto.php

sig: 9908432d8bddb847a25c45fce00656ab
login_type: 1
openid: openid_23903
channel: channel_0018442
version: V.2.9
device_id: A00-B1-C24-D170
login_num size: 10
login_num: 1,2,3,4,5,6,7,8,9,10
```
