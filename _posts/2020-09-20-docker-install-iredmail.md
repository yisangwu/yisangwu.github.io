---
layout: post
title: docker 安装iredmail邮件服务器
tags: php, socket
categories: php
---


iredmail官方docker镜像地址：
https://hub.docker.com/r/iredmail/mariadb

> 可使用的镜像：
>* iredmail/mariadb:stable: Stable version. （镜像的linux系统是Alpine Linux 3.12）
>* iredmail/mariadb:nightly: Triggered by EACH GitHub commit.

#### 安装docker，使用官方安装脚本自动：
```shell
$curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
#### 修改docker镜像源：
```shell
$vim /etc/docker/daemon.json
    {"registry-mirrors":["https://reg-mirror.qiniu.com/"]}
```
#### 重新加载systemctl服务配置文件：
```shell
$systemctl daemon-reload
```
#### 启动 Docker：
```shell
$systemctl start docker
```
#### 拉取iredmail镜像：
```shell
$docker pull iredmail/mariadb:stable
```
### 创建iredmail目录：
```shell
$mkdir /iredmail
```
#### 进入目录，创建子目录，docker目录映射使用：
```shell
$cd /iredmail
$mkdir -p data/{backup,clamav,custom,imapsieve_copy,mailboxes,mlmmj,mlmmj-archive,mysql,sa_rules,ssl,postfix_queue}
```
#### 创建iredmail配置文件：
```shell
$vim iredmail-docker.conf
    HOSTNAME=访问后台和登录邮箱的域名(mail.abc.com) >> iredmail-docker.conf
    FIRST_MAIL_DOMAIN=默认登录邮箱域名(abc.com) >> iredmail-docker.conf
    FIRST_MAIL_DOMAIN_ADMIN_PASSWORD=iredadmin的管理员密码 >> iredmail-docker.conf
    MLMMJADMIN_API_TOKEN=$(openssl rand -base64 32) >> iredmail-docker.conf
    ROUNDCUBE_DES_KEY=$(openssl rand -base64 24) >> iredmail-docker.conf
    MYSQL_ROOT_PASSWORD=数据库root密码 >> iredmail-docker.conf
```

#### 启动iredmail容器：
>> 官方容器启动使用 docker run --rm ... ,不是后台运行容器，终端断开会自动退出释放容器。
>> docker run 命令带 --rm命令选项，等价于在容器退出后，执行 docker rm -v。
>> 注意在公有云，安全组要放开容器端口的 0.0.0.0 全部可访问。
>> 如果要邮箱支持https://mail.abc.com/mail/, 开启443端口。
>> 如果仅支持http，需要修改  iredadmin， roundcubemail的强制https重定向。

```shell
$docker run \
    -d \
    --name iredmail \
    --env-file iredmail-docker.conf \
    --hostname mail.abc.com \
    -p 80:80 \
    -p 443:443 \ 
    -p 110:110 \
    -p 995:995 \
    -p 143:143 \
    -p 993:993 \
    -p 25:25 \
    -p 465:465 \
    -p 587:587 \
    -v /iredmail/data/backup:/var/vmail/backup \
    -v /iredmail/data/mailboxes:/var/vmail/vmail1 \
    -v /iredmail/data/mlmmj:/var/vmail/mlmmj \
    -v /iredmail/data/mlmmj-archive:/var/vmail/mlmmj-archive \
    -v /iredmail/data/imapsieve_copy:/var/vmail/imapsieve_copy \
    -v /iredmail/data/custom:/opt/iredmail/custom \
    -v /iredmail/data/ssl:/opt/iredmail/ssl \
    -v /iredmail/data/mysql:/var/lib/mysql \
    -v /iredmail/data/clamav:/var/lib/clamav \
    -v /iredmail/data/sa_rules:/var/lib/spamassassin \
    -v /iredmail/data/postfix_queue:/var/spool/postfix \
    iredmail/mariadb:stable
```

#### 附录：

* 端口说明：
```
    80: HTTP
    443: HTTPS
    25: SMTP
    465: SMTPS (SMTP over SSL)
    587: SUBMISSION (SMTP over TLS)
    143: IMAP over TLS
    993: IMAP over SSL
    110: POP3 over TLS
    995: POP3 over SSL
    4190: Managesieve service
```

1.  iredmail管理后台：
    - 管理后台地址：https://mail.abc.com/iredadmin (mail.abc.com, HOSTNAME 在iredmail-docker.conf 中配置)
    - 管理员账号：postmaster@ucloudgame.com
    - 管理员密码：FIRST_MAIL_DOMAIN_ADMIN_PASSWORD (在iredmail-docker.conf 中配置)

2.  在iredmail管理后台:
    - 使用当前域（abc.com）创建用户(指定密码)。
    - 添加新的域，再添加此域的用户。
  
3.  iredmail 邮箱登录主页：
    - 登录地址：https://mail.abc.com/mail
    - 邮箱账号：xxx@abc.com （iredadmin 创建指定域下面的用户名）
    - 邮箱密码：123 （iredadmin 创建用户时指定）
  
4.  自定义邮箱登录页面底部水印：
```php 
/opt/www/roundcubemail/config/config.inc.php:
$config['product_name'] = 'Tencent.com Webmail';
```
