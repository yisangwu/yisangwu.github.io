---
layout: post
title: TOA - TCP Option Address
tags: CentOS tcp toa
categories: CentOS
---
<style type="text/css">
    p{text-indent: 20px}
</style>

<p>使用Haproxy转发TCP到业务服务器端，后端报文看到的源 IP 地址是代理服务器的IP 。为了让后端能够获取到用户端实际的 IP 地址，有三个方案：</p>

### 1. http 请求记录标识：
<p>在客户端和服务器建立TCP连接之前，一般都是先http请求拿到服务器的IP和PORT。在http请求时，使用客户端的ID或者macid，device_id，golbal_id等唯一标识映射用户的ip，存储起来，ex：redis的hash 或 zset。</p>
<p>这样，后端tcp服务器就可以根据唯一标识，在存储中拿到用户的ip。虽然不是最精准的，但是也是最接近真实的。</p>

### 2. haproxy 配置添加send-proxy：
ex：
```shell
listen tcp-frontend
    bind 192.168.123.77:8577
    balance roundrobin
    mode tcp
    option tcplog
    server tcp-backend 192.168.123.60:8577 send-proxy 
```
<p>加了send-proxy后，第一次socket连接时，服务端会收到proxy协议： </p> 
```shell  
PROXY TCP4 192.168.123.55 192.168.123.77 52505 8577 
PROXY TCP6 ... .. ... ... 
#PROXY TCP4/6 client_ip, proxy_ip, client_port, proxy_port
```
<p>里面有haproxy的Ip与用户真实IP。改造下tcp服务器，就可以拿到客户端tcp连接时的真实ip。</p>

### 3. TOA（TCP Option Address）：
<p>TCP 协议下，为了将客户端 IP 传给服务器，会将客户端的 IP，port 在转发时放入了自定义的 tcp option 字段。</p>
#### [华为云](https://www.huaweicloud.com/)的弹性负载均衡（ELB）：    
<https://support.huaweicloud.com/eu-west-0-usermanual-elb/zh_cn_elb_06_0001.html>  
TOA模块源代码地址：  
&nbsp;&nbsp;&nbsp;&nbsp;<https://github.com/Huawei/TCP_option_address>

#### [腾讯云](https://www.huaweicloud.com/)的宙斯盾安全防护：  
<https://cloud.tencent.com/document/product/685/18805>  
TOA内核安装包地址：  
&nbsp;&nbsp;&nbsp;&nbsp;(1) [Centos 6.x 下载](http://toakernel-1253438722.cossh.myqcloud.com/kernel-2.6.32-220.23.1.el6.toa.x86_64.rpm?_ga=1.63957738.456440398.1588862943)  
&nbsp;&nbsp;&nbsp;&nbsp;(2) [Centos 7.x 下载](http://toakernel-1253438722.cossh.myqcloud.com/kernel-3.10.0-693.el7.centos.toa.x86_64.rpm?_ga=1.1904372.456440398.1588862943)

