---
layout: post
title: CentOS7 下部署Google Authenticator
tags: centos ssh google
categories: security
---
<style type="text/css">
    p{text-indent: 20px}
</style>
> 环境：CentOS7

### Google Authenticator
<p>Google身份验证器是一款基于时间与哈希的一次性密码算法的两步验证软件令牌，此软件用于Google的认证服务。此项服务所使用的算法已列于 RFC 6238 和 RFC 4226 中。</p>

查看当前yum源有无Google Authenticator：
```shell
$ yum list | grep authenticator
```

Linux (apk, rpm, tgz)下载地址：
<https://pkgs.org/download/google-authenticator>

#### 1. 下载Google Authenticator对应服务器版本的rpm软件包：
$ wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/g/google-authenticator-1.04-1.el7.x86_64.rpm

#### 2. 安装：
$ rpm -ivUh google-authenticator-1.04-1.el7.x86_64.rpm

#### 3. 查看是否安装：
$ yum list installed  | grep google

#### 4. 添加google_authenticator认证模块：
 - 查找google_authenticator库：
```shell
$ find / -name 'pam_google_authenticator.so'  
/usr/lib64/security/pam_google_authenticator.so
```
 - sshd添加Google Authenticator认证：
```shell
$ vim /etc/pam.d/sshd
  # 第一行添加
  auth required pam_google_authenticator.so
```
如果你希望没有开启两步验证的账号还可以登录，则在此文件后面添加nullok：
```shell
auth required pam_google_authenticator.so nullok
```
最后一行追加，nullok表示PAM方法是可选的，这允许没有OATH-TOTP令牌的用户仍然使用他们的SSH密钥登录。 一旦所有用户都有OATH-TOTP令牌，您可以从此行中删除nullok ，以使MFA成为强制性。


#### 5. sshd添加pam认证：
```shell
$ vim /etc/ssh/sshd_config
  ChallengeResponseAuthentication yes
```

#### 6. 切到需要使用Google身份验证器的账号，执行命令：
```shell
$ google-authenticator
```
> 手机安装google authenticator， 在google_authenticator软件中扫描服务器上面的二维码，添加认证信息配置。
>>注意保存二维码图片，和紧急备用code

#### 7. root重启sshd服务，使配置生效：
```shell
$ service sshd restart
```
再次ssh登录服务器，则会出现提示，输入code（手机安装Google Authenticator对应用户@服务器的动态验证码），验证通过后再次输入账号的密码。密码正确，才可以进入服务器。

