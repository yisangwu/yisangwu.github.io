---
layout: post
title: git 配置多个ssh key账户
tags: git CentOS
categories: git
---
<style type="text/css">
    p{text-indent: 20px}
</style>
<p>近来，除了github外，还有其他的代码托管平台。配置http/https的远程仓库地址，在拉取提交等操作时，经常要输入用户名，密码。因为ssh配置ssh keys后，就是免密操作。所以，更换git的远程仓库地址为ssh。（当然http方式的免密登录，可以使用.git-credentials 配置。）</p>
<p>但是，有多个git仓库平台（主要是指host不同），像公司代码仓库，github仓库等，如果都要配置ssh keys就要解决rsa key覆盖的问题。</p>


####  本地配置多个git代码平台的ssh keys：
> 环境：windows， 终端 Git Bash，已有ssh key的情况下，再次添加github 的ssh key。  

```shell
$ ll  # 当前环境已存在ssh key
-rw-r--r-- 1 yisan 197609 3243 11月  22  2017 id_rsa   
-rw-r--r-- 1 yisan 197609  742 11月  22  2017 id_rsa.pub
-rw-r--r-- 1 yisan 197609 3741 11月  22 22:06 known_hosts
```

1. 生成github.com 邮箱账号对应的ssh key，并命名为github_rsa：
```shell
$ ssh-keygen -t rsa -C "yisangwu@hao123.com" -b 4096 -f ~/.ssh/github_rsa
```
一路回车，成功生成ssh key。在  ~/.ssh 目录下查看：
```shell
$ ll
-rw-r--r-- 1 yisan 197609  136 11月  22 23:37 config
-rw-r--r-- 1 yisan 197609 3381 11月  22 23:31 github_rsa
-rw-r--r-- 1 yisan 197609  742 11月  22 23:31 github_rsa.pub
-rw-r--r-- 1 yisan 197609 3243 11月  22  2017 id_rsa
-rw-r--r-- 1 yisan 197609  742 11月  22  2017 id_rsa.pub
-rw-r--r-- 1 yisan 197609 3741 11月  22 22:06 known_hosts
```
2. 配置ssh key的路由策略config：
  - 在 ~/.ssh 目录下，创建config文件：
```shell
$ touch ~/.ssh/config
```
  - 编辑config文件，添加github.com的路由策略：
```shell
$ vim ~/.ssh/config
# github
Host github.com
    HostName github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa  # github账号生成的ssh key 私钥
```
3. 将公钥 github_rsa.pub 添加到 github的settings/keys下。
4. 测试ssh是否配置成功：
```shell
$ ssh -T git@github.com  # -T 不显示终端，只显示连接成功信息
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
```
5. 修改github仓库的远程地址为ssh地址：
```shell
$ git remote set-url origin git@github.com:<hello>/<xxx>.git
```
6. 拉取远程仓库代码：
```shell
$ git pull
Already up to date.
```

至此，配置成功。