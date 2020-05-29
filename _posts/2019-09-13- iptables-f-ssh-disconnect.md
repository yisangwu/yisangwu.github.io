---
layout: post
title: iptables-F后SSH连接断开
tags: linux CentOS iptables
categories: CentOS
---

最近回收利用一台被征用做邮件服务的服务器，重新部署新的业务。

清理了所有的安装软件和目录文件后，调整了网络安全组规则，仅开放所需端口。

禁用selinux后，看了下防火墙的配置：

```shell
# iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED 
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     icmp --  anywhere             anywhere            icmp echo-request 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:ssh 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:http 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:https 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:smtp 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:pop3 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:pop3s 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:imap 
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:imaps 
...
...

Chain FORWARD (policy DROP)
target     prot opt source               destination         
...
...
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination



```

部署业务服务的时候，发现对应的tcp端口，除了本机127.0.0.1,，内外网都不能访问！

确认，网络安全组规则已经开放了公网访问。

更换端口也不可以访问。

这就很奇怪了。

想到iptables的规则，为避免干扰，先清理下：

```shell
# iptables -F
```

结果，xshell立即断开。 怎么重试，都不能连接。 走服务器所在内网机器，ssh也不能连接。

> 附：服务器在Azure，系统为 CentOS6.10 Final。

从Azure 的控制台，可以从虚拟机的串行控制台连接上。

但是Azure的串行控制台，日志刷屏太厉害了，不仅看不到命令执行的结果，连输入命令都是很困难的事情。

想了个招，登录进去，执行命令查看了sshd_config，hosts.allow，hosts.deny等相关的设置， 都是正常的。

剩下的就是iptables的规则是清空的。

这下就凉凉了。Azure 不像 AWS，阿里云，腾讯云，百度云，华为云等，可以在OS盘上，重装系统。  
可以操作的只有：

-   尝试重新部署
-   有备份，进行还原；
-   交换OS系统盘；
-   重新建一个虚拟机；

尝试重新部署，ssh连接恢复正常。

重新部署后，除了临时硬盘的数据会全部丢失，其他虚拟机的配置，数据等都保持不变。

再次查看 服务的tcp端口，还是一样，各种都不能访问。

发现iptables又是一模一样的。 业务部署的TCP端口，内外网还是不能访问。

顿时，好奇心来了，再次清空iptables：

```shell
# iptables -F
```

结果，ssh 立刻就断开了。 又是内外网都不能连接。

瞬间老实了，立刻进串行控制台，添加iptables的ssh端口规则，网络安全组已经开放22端口：

```shell
# iptables -I INPUT -p tcp --dport 22 -j ACCEPT
```

ssh 立即恢复正常。

仔细瞅瞅iptables 的规则，发现INPUT链路的策略是**DROP** ：  
Chain INPUT (policy DROP)

修改INPUT策略为ACCEPT：

```shell
# iptables -P INPUT ACCEPT
```

再次清空iptables：

```shell
# iptables -F
```

发现：  
1.ssh没有断开连接，内外网ssh都可以正常访问；  
2\. 服务部署的tcp端口，可以正常访问；