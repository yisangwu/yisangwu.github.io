---
layout: post
title: CentOS7 下git源码升级版本
tags: git CentOS
categories: git
---
<style type="text/css">
    p{text-indent: 20px}
</style>

服务器上的git版本太低了，还停留在古老的版本1.8.3.1，今天抽空升级了版本。

git官方源码下载地址：
https://www.kernel.org/pub/software/scm/git/

### 升级步骤, root权限操作：

1. 查看本机安装的git版本，默认安装或使用yum安装： 
```shell
$ git version
git version 1.8.3.1
```
注：如找不到git命令，那就直接源码安装git了。 使用yum安装的版本不高。yum安装命令为：
```shell
$ yum install -y git
```
2. 下载源码：
```shell
$ wget https://www.kernel.org/pub/software/scm/git/git-2.14.1.tar.gz
```
3. 解压并进入文件夹：
```shell
$ tar zxf git-2.14.1.tar.gz && cd git-2.14.1
```
4. 检查环境依赖与配置：
```shell
$ ./configure
```
5. 编译并且安装：
```shell
$ make && make install
```
6. 检查git版本：
```shell
$ git --version
```
7. git版本号不是安装的版本号，则查找git安装的路径：
```shell
$ whereis git
```
8. 添加软连接：
```shell
$ ln -s /usr/local/bin/git /usr/bin/git
```
9. 再次检查git版本号，完成源码升级（源码安装）：
```shell
$ git version
```