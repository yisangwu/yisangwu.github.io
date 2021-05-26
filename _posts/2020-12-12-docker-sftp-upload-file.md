---
layout: post
title: docker 容器支持SFTP上传文件
tags: docker, DevOps
categories: linux
---


### 使用镜像启动容器：
```bash
$ docker run -itd --restart=always --privileged=true --net='bridge' --name kooi -p 80:80 -p 25770:25770 -p 22000:22 192.168.0.77:5000/mmox:env /sbin/init
```
>注：
>> -p: 指定端口映射，格式为：主机(宿主)端口:容器端口  
>> -p 22000:22 将宿主机的22000映射到容器的22 端口 ，内网SFTP上传代码到容器里面  
>> ( 或者使用 -v 目录映射，直接SFTP上传文件到宿主机目录下）

### 容器内安装ssh服务：
#####  1. 进入docker：
```bash
 $ docker exec -it kooi /bin/bash
```
##### 2. 安装ssh服务: 
```bash
$ yum install -y openssh-server
```
##### 3. 启动ssh服务:
```bash
 $ systemctl start sshd
```
##### 4. 查看22端口SSH服务是否启动:
```bash
# netstat -lnpt | grep 22
tcp   0      0 0.0.0.0:22    0.0.0.0:*    LISTEN   717/sshd  
tcp6  0      0 :::22            :::*      LISTEN   717/sshd 
```
##### 5.  创建FTP用户:
```bash
 $ useradd -d /home/user_ftp -g users -m user_ftp
```
###### 6. 设置FTP用户的密码： 
```bash
$passwd user_ftp
```
### SFTP登录宿主机，进入docker：
SFTP连接宿主机 ip:22000，用户user_ftp登录docker，上传文件。
> 如果宿主机有多个docker环境，使用端口区分开 
>> ex:  docker run ...  -p 22000:22  
>> ex:  docker run ...  -p 22001:22


------------

> 注：
>> 网上有教程，容器启动后还可以改端口映射，  
>> ex， 修改hostconfig.json，config.v2.json等文件。  
>> 是不科学的。  
>> 最快的办法是docker commit新构镜像，用这个新的镜像重起一个容器。  
>> 最硬核的办法是删掉容器，检查好端口映射，启动一个新容器。
