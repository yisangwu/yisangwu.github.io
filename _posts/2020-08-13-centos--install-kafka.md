---
layout: post
title: CentOS下安装Kafka==2.6.0
tags: centos, kafka
categories: centos
---

> 官方地址: http://kafka.apache.org/quickstart  
> 中文文档: https://kafka.apachecn.org/  

### 1. 下载kafka源码：
```shell
# wget https://mirror.bit.edu.cn/apache/kafka/2.6.0/kafka_2.13-2.6.0.tgz
````
### 2. 解压到指定目录：
```shell
# tar -C /usr/local/ -zxf kafka_2.13-2.6.0.tgz
```
### 3. 进入源码目录：
```shell
# cd /usr/local/kafka_2.13-2.6.0/
```
### 4. 启动ZooKeeper服务：
Kafka 使用 ZooKeeper，需要先启动一个ZooKeeper服务器。 创建一个单节点ZooKeeper实例:
```shell
# ./bin/zookeeper-server-start.sh ./config/zookeeper.properties
```

### 5. 启动Kafka服务器:
```shell
# ./bin/kafka-server-start.sh ./config/server.properties
```

### 6.后台运行:

#### 6.1 使用 daemon参数：
```shell
# ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties
# ./bin/kafka-server-start.sh -daemon ./config/server.properties
```
#### 6.2 使用nohup:
```shell
# nohup ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties >/dev/null 2>&1 &
# nohup ./bin/kafka-server-start.sh -daemon ./config/server.properties >/dev/null 2>&1 &
```
