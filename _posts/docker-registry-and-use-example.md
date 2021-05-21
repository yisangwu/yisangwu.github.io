---
layout: post
title: docker Registry部署私有仓库及实用案例
tags: docker, registry
categories: linux
---


为统一环境部署，减少linux开发环境的重复安装Nginx，Mysql，MongoDB，Redis，RabbitMQ，PostgreSQL，尤其是Python版本和项目依赖第三方库。

在内网搭建一个私有的docker仓库，将几个项目的开发环境都做好镜像推到私有仓库，供开发下载部署。

###### registry 官方文档地址：[https://docs.docker.com/registry/](https://docs.docker.com/registry/)

> 仓库机器：部署registry存储docker镜像的仓库机器。  
> 镜像机器：制作镜像，推到私有仓库的机器。  
> 客户端机器：从私有仓库拉取镜像，部署docker容器的机器。

## 仓库机器

-   安装好docker，配置docker镜像仓库地址，拉取registry镜像：

```bash
$ docker pull registry
```

-   启动registry:

```bash
$ docker run -i -t -d --restart=always \
--privileged=true \
--name docker-hub \
-p 5000:5000 \
-v /docker-hub:/var/lib/registry \
registry:latest
```
> 复制使用，请转成一行
-   查看容器是否启动：
```bash
$ docker ps -a
```
## 镜像机器
- 安装好docker， docker配置非 https 仓库地址 :
> 需要访问私有仓库的机器都需要配置私有仓库地址
```bash
$ vim /etc/docker/daemon.json
 {
  "registry-mirror": [
    "https://reg-mirror.qiniu.com/",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "insecure-registries": [
    "192.168.0.88:5000"
  ]
}
```
- 重新加载docker配置：
```bash
$ systemctl daemon-reload
```
- 重启docker：
```bash
$ systemctl restart docker
```
- 拉取 centos8 docker镜像：
```bash
$ docker pull centos:centos8
```
- 启动centos8 容器：
```bash
$ docker run -itd --net='bridge' --privileged=true --name centos8 centos:8 /sbin/init
```
- 进入容器：
```bash
$ docker exec -it centos8 /bin/bash
```
- 安装项目需要的软件和Python及依赖库
```bash
$ 略
```
- 环境部署完成后，将容器保存为新的镜像,并添加提交人信息和说明信息：
```bash
$ docker commit -a "ooc" -m "env" 容器ID  gServer:v1
```
- 标记本地镜像，打个tag：
> 如果需要调整部署，直接在容器488406ff141b中调整，重新打tag。
```bash
$ docker tag gServer:v1 192.168.0.88:5000/gServer:v1
```
- 将镜像上传到镜像仓库
```bash
$ docker push 192.168.0.88:5000/gServer:v1
```
- 查询镜像:
```bash
$ curl 192.168.0.88:5000/v2/_catalog
{"repositories":["gServer"]}
```
- 查询镜像tag(版本):
```bash
$ curl 192.168.0.88:5000/v2/gServer/tags/list
{"name":"gServer","tags":["v1"]}
```

# 客户端机器
- 安装好docker， docker配置非 https 仓库地址 :
> 需要访问私有仓库的机器都需要配置私有仓库地址
```bash
$ vim /etc/docker/daemon.json
 {
  "registry-mirror": [
    "https://reg-mirror.qiniu.com/",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "insecure-registries": [
    "192.168.0.88:5000"
  ]
}
```
- 重新加载docker配置：
```bash
$ systemctl daemon-reload
```
- 重启docker：
```bash
$ systemctl restart docker
```
- 从私有仓库，拉取镜像：
```bash
$ docker pull 192.168.0.88:5000/gServer:v1
```
- 启动容器：
```bash
$ docker run -itd --privileged=true --net='bridge' --name gServerA -p 8080:80 -p 17650:17650 192.168.0.88:5000/gServer:v1 /sbin/init
```
-   查看容器是否启动：
```bash
$ docker ps -a
```
- 进入容器，查看制作镜像时安装的东西是否都在：
```bash
$ docker exec -it gServerA /bin/bash
```
