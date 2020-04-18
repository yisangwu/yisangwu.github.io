---
layout: post
title: iRedMail 邮件服务器配置多域名
tags: linux centos iRedMail multi domain
categories: iRedMail 
---


iRedMail 主要使用的后台系统有两个：

1. iredadmin （管理域名和邮箱账户）：
    * python开发，使用web.py框架；
    * 地址：https://mail.domain.com/iredadmin/；

2. roundcubemail（Web邮件系统，类似mail.qq.com, mail.163.com）：
    * php开发，无框架；
    * 地址：https://mail.domain.com 访问会重定向到 https://mail.domain.com/mail；
    


两个后台默认安装在/opt/www/ 目录下, 可以看到对应的版本：
```shell
$ ll
Feb 68 88:88 iredadmin -> /opt/www/iRedAdmin-1.0
Feb 68 88:88 iRedAdmin-1.0
Feb 68 88:88 roundcubemail -> /opt/www/roundcubemail-1.4.2
Feb 68 88:88 roundcubemail-1.4.2
```

刚好php，python 都略懂一点点，为了排查两个问题：
1. iredadmin登录不了，使用的是安装时配置的 postmaster 账号；
2. roundcubemail支持多个域名，为实现一次部署，多域名使用；

看了下两个系统的源码，大致找到了解决的办法。

iRedAdmin，管理员账号登陆失败，排查思路： 
 1. 查看settings.py 中配置的 admin 邮箱账号 和 对应使用的后端存储backend；
 2. 找到backend对应的路由，libs/iredbase.py 引入的url配置（如settings.backend=mysql，则路由为：controllers.mysql.urls）；
 3. 登录路由login，对应 controllers.mysql.basic.Login；
 4. 查找登录认证调用的auth方法，在libs/mysql/core.py中；
 5. 查看mysql的vmail库，先找admin表，再找mailbox表，查找 admin 账号对应的记录；

----------
roundcubemail 是支持多域名登录, 但是默认是未开启。 Baidu 无解， 翻墙 Google 也不胜明了，索性看源码。
```php
/*
 不支持多域名登录:
  a. 设置此配置，登录时使用此域名，只用填用户名;
  b. 未设置此配置，则需要完整的用户名带邮箱后缀才可以登录;
*/
$config['username_domain'] = 'a.com';
```

支持多域名登录, 只需要修改配置 /opt/www/roundcubemail/config/config.inc.php，

* <span style="color:red">多域名对应不同的邮件服务器</span>，即域名指向的公网IP不同 (也可以一个邮件服务器绑定多个公网IP）：  
    ```php
    // 支持多域名登录
    // 登录时默认使用的域名，只用填用户名
    $config['username_domain'] = [
       '1.2.3.4' => 'a.com',
       '5.6.7.8' => 'b.com',
    ];

    // 登录选择host, select 标签key是host，value是对应显示值；
    $config['default_host'] = array(
       '1.2.3.4' => 'a.com',
       '5.6.7.8' => 'b.com',
    );
    ```
    更简单的写法：
    ```php
    // 支持多域名登录
    $config['username_domain'] = $config['default_host'] = array(
       '1.2.3.4' => 'a.com',
       '5.6.7.8' => 'b.com',
    );
    ```

* <span style="color:red">一个邮件服务器绑定多个域名</span>，开启多域名登录：
    > 1. 部署一个邮件服务器，多个域名使用。节省机器同时节省时间；  
    > 2. a.com, b.com, c.com 都指向邮件服务器的同一个ip；
  
  * 修改配置 /opt/www/roundcubemail/config/config.inc.php，
    ```php
    $config['username_domain'] = $config['default_host'] = array(
       '127.0.0.1' => 'a.com',
       '127.0.0.2' => 'b.com',
       '127.0.0.3' => 'c.com',
    );
    // 使用的ip如127.0.0.1 或 127.0.0.2， 只要ip格式正确，可以随便写。
    ```


  * 重点中的重点，是修改 /opt/www/roundcubemail/program/include/rcmail.php：
    ```php
    $host = '127.0.0.1';  // 加上host重置
    $host = rcube_utils::idn_to_ascii($host);  //原代码
    ```

#### roundcubemail 配置支持多域名后，还有**两个**操作:
  1. b.com, c.com 的域名解析添加 MX记录指向 mail.a.com:
  2. 管理员账号，登录iredadmin，添加域 和 域下对应的用户邮箱账号并分配空间；

----------
至此，才真正实现：
  * 访问 mail.b.com/mail.c.com 会跳转到mail.a.com；
  * 直接在mail.a.com 上面，登录 b001@b.com, c001@c.com 账号，收发邮件；

    > 注：
    > > 修改 roundcubemail 代码，不需要重启服务，直接刷新页面；  
    > > 修改 iredadmin 代码，则需要重启 uwsgi 服务；