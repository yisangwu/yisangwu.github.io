---
layout: post
title: iRedMail 邮件服务器部署
tags: linux centos iRedMail
categories: iRedMail 
---

<style type="text/css">
    dd{text-indent: 20px}
</style>

> 注： www.iredmail.org 或 www.iredmail.com  

* 官网： https://www.iredmail.org/index-zh_CN.html
* 下载地址： https://www.iredmail.org/download.html
* 中文文档地址： https://docs.iredmail.org/index-zh_CN.html  

#### 按照文档选择对应的环境安装。

> 一个比较好的习惯：在screen分屏下安装iredmail，避免因网络突然掉线或者终端被关掉等等原因，造成安装过程被中断。  

``` shell
$ screen -S 会话名   #　创建会话，使用root，因为是root权限安装
```

#### 安装注意事项：

1. 禁用 SELinux：
  * 永久关闭Selinux,：  
  ```shell
  root权限修改/etc/selinux/config，设置SELINUX=disabled（需要重启机器生效配置）。
  ```
  * 临时关闭Selinux，服务器重启后失效：
    ```shell
    $ setenforce=0 
    ```
2. 服务器环境没有部署安装iredmail相关的组件，例如Ngin、Mysql、Apache、OpenLDAP、postfix、Dovecot、Amavisd等， 避免冲突。
3. 邮箱域名需要ssl证书（单域名证书或通配符证书）。iredmail默认域名要支持https，在Nginx环境下,配置了强制http转https。不要https支持，可以修改Nginx配置（不建议）。
4. **<span style="color:red">重点: </span>**安装时，添加邮件服务器域名时，注意是 abc.com，一定不要填 *.abc.com 或 mail.abc.com

----------
#### 域名添加DNS解析,A(主机地址)记录，MX(邮件交换器)记录：
例如, a.com 添加 mail.a.com 解析( 优先级，默认10 ):  
<table>
    <tr>
        <th>类型</th>
        <th>名称</th>
        <th>值</th>
    </tr>
    <tr>
        <td>A</td>
        <td>mail</td>
        <td>服务器公网IP</td>
    </tr>
    <tr>
        <td>MX</td>
        <td>@</td>
        <td>mail.a.com</td>
    </tr>    
</table>

多个域名的指向同一台机的同个公网IP，此时解析，只需要添加MX记录：

例如, mail.b.com 使用 mail.a.com 的邮件服务器，则只需在b.com的DNS解析上，添加MX记录( 优先级，默认10 )：
<table>
    <tr>
        <th>类型</th>
        <th>名称</th>
        <th>值</th>
    </tr>
    <tr>
        <td>MX</td>
        <td>@</td>
        <td>mail.a.com</td>
    </tr>    
</table>


