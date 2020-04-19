---
layout: post
title: CentOS7 下gitlab修改域名host
tags: git CentOS gitlab
categories: git
---
<style type="text/css">
    p{text-indent: 20px}
</style>
<p>因原git域名用作他途，需要更新gitlab的域名。 找了个周末的凌晨更换了域名。</p>

准备工作：
>1. 新域名的DNS解析，添加 A(主机地址)记录，指向gitlab服务器。
>2. 更新域名时，一定不要有代码提交操作。

<p>更新域名比较简单，只要restart成功，gitlab的所有服务都正常重启，就没有问题了。</p>

### 更新步骤如下, root权限操作：

1.修改gitlab.rb文件中，域名相关的地方：
```shell
$ grep 'xx.com' /etc/gitlab/gitlab.rb
external_url 'http://git.xx.com'
gitlab_rails['gitlab_ssh_host'] = 'git.xx.com'
gitlab_rails['gitlab_email_from'] = 'gitlab@xx.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@xx.com'
user['git_user_email'] = 'gitlab@.xx.com'
```
2.更新配置gitlab.yml文件：
>注意：不需要单独去修改gitlab.yml文件，网上很多的文章都写着要修改，其实不用。 

```shell
$ gitlab-ctl reconfigure
```
3.重启GitLab服务：
```shell
$ gitlab-ctl restart
```
4.查看gitlab的所有服务状态：
```shell
$ gitlab-ctl status
```
5.登录gitlab，可以看到对应仓库的ssh/http地址已经更新为新的域名。  

如不能正常访问，先排查DNS的解析，再看gitlab日志。
查看gitlab日志：
```shell
$ gitlab-ctl tail
```
