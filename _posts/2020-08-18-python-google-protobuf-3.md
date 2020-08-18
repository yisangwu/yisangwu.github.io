---
layout: post
title: Python使用 Google Protocol Buffers (protobuf)
tags: python, protobuf
categories: python
---
最近在用Python写游戏服务器，想更换掉老旧的protobuf2，  
看了下proto3的语法，来测试下：

## 一、安装 Python 的protobuf包：

### 1.1 默认安装最新版本
```shell
# pip install protobuf
```
### 1.2 查看protobuf包信息：
```shell
# pip show protobuf
Name: protobuf
Version: 3.12.2
Summary: Protocol Buffers
Home-page: https://developers.google.com/protocol-buffers/
Author: None
Author-email: None
License: 3-Clause BSD License
Location: /usr/local/lib/python3.6/site-packages
Requires: setuptools, six
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
# mkdir proto_python
```

### 2.3 编译proto文件，生成python类：
```shell
# protoc --python_out=./proto_python  pack.proto
```
>> 目录结构如下：
```shell
─proto_python
    │  pack_pb2.py
    │  __init__.py
    │
    └─__pycache__
├─ test_proto.py
```

## 三、测试脚本：

```python
# cat test_proto.py
# coding=utf8

"""
protobuf 测试, python

编译proto文件，生成 python类
# protoc --python_out=./proto_python pack.proto
安装protobuf解析第三方包，google出品
# pip install protobuf

"""
import uuid
import time
import random
import string
from proto_python.pack_pb2 import LoginType, LoginReq


# 实例化
req = LoginReq()
req.sig = uuid.uuid4().hex

# 枚举变量
req.login_type = LoginType.WEICHAT
req.openid = 'openid_{}'.format(int(time.time()))
req.channel = '-'.join(random.sample(string.ascii_uppercase, 6))
req.version = 'v{}'.format(random.randint(10, 200))

random_list = string.ascii_lowercase + string.ascii_uppercase + string.digits
req.device_id = ''.join(random.sample(random_list, 25))

req.mac_id = '-'.join(random.sample(string.ascii_lowercase +
                                    string.digits, 10))

# repeated 的两种填充方式，和 protobuf2 不同
req.login_num.append(1)
req.login_num.extend([2, 3, 4])

# 序列化成二进制字符串
data = req.SerializeToString()

# 解析实例化
req_parse = LoginReq()
# 二进制字符串反序列化
req_parse.ParseFromString(data)

print(type(req_parse), req_parse)
print(req_parse.sig, req_parse.login_type,
      req_parse.openid, req_parse.login_num)
```

输出结果如下：
```shell
# python test_proto.py
<class 'pack_pb2.LoginReq'> sig: "e8ab25f6f1474cb7b2e2c85f61fdad12"
login_type: WEICHAT
openid: "openid_1597743198"
channel: "S-N-T-A-E-L"
version: "v163"
device_id: "UP5xYgMdOlEvXsezGc23QNua6"
mac_id: "c-7-f-q-y-j-8-5-s-g"
login_num: 1
login_num: 2
login_num: 3
login_num: 4

e8ab25f6f1474cb7b2e2c85f61fdad12 7 openid_1597743198 [1, 2, 3, 4]
```
