---
layout: post
title: git 使用中遇到的一些问题
tags: git CentOS
categories: git
---
<style type="text/css">
    p{text-indent: 20px}
</style>
### [SmartGit](https://www.syntevo.com/smartgit/download/)  
是git的一种很好用的可视化工具，收费的。可以注册非商业用途免费使用。[注册地址](https://www.syntevo.com/cn/smartgit/register-non-commercial)
><p>注意，使用git仓库用户的邮箱注册。如果注册邮箱和git仓库的用户邮箱不一致，会出现弹窗。</p>

#### >>  使用中的一些问题：

1. 使用SSH keys 免密操作git：
   - linux使用xshell等SSH客户端
   - windows使用Git Bash：
```shell
   $ ssh-keygen -t rsa -C "yisangwu@hao123.com"  -b 4096  # 使用git账号生成密钥
   $ cat ~/.ssh/id_rsa.pub  # 复制公钥，添加到个人设置的ssh key里面。
```
2. 本地项目，关联git远程仓库:
```shell
$ git init  # 初始化一个新本地仓库
$ git remote add origin git_url  # 建立和远程仓库的连接
$ git remote -v  # 查看远程地址
$ git pull origin master  #　拉取远程master分支
```
3. 修改git远程仓库地址:
```shell
$ git remote set-url origin new_git_url
```
4. 拉取远程代码报错，fatal: refusing to merge unrelated histories：
```shell
#　branch_name　对应远程分支名
$ git pull origin branch_name --allow-unrelated-histories  
# 再进行 git 操作
```
5. git clone --depth=1 远程分支看不到的问题：
> 有些git仓库代码过大，达到几个G，初始化clone或者fetch分支时，经常timeout，或者bad header。即使修改了git的配置，还是会出现拉取失败！！

```shell
# depth指定克隆深度,为1即表示只克隆最近一次commit
$ git clone --depth 1 https://git123.com/group_name/coder.git
$ git remote -r   #查看远程分支, 此时看到的只有master
$ git remote set-branches origin remote_branch_name  # 添加远程分支
$ git fetch --depth 1 origin remote_branch_name  # fetch远程分支，深度为1
$ git checkout remote_branch_name
```
6. git忽略文件权限修改，避免修改文件权限导致的文件冲突或修改：
```shell
$ git config core.filemode false  # 在仓库目录下执行
```
7. 远程分支不存在，但是本地还有：
```shell
$ git pull -p  # 在仓库目录下执行
```
