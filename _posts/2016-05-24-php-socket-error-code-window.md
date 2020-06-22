---
layout: post
title: window下 PHP socket 错误码预定义常量， 对应数值，错误信息
tags: php, socket
categories: php
---

>php版本：
>>PHP 7.4.0 (cli) (built: Nov 27 2019 10:14:18) ( ZTS Visual C++ 2017 x64 )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Xdebug v2.9.2, Copyright (c) 2002-2020, by Derick Rethans

|错误码预定义常量|  数值| 错误信息|
| :-----| :-----| :-----|
|SOCKET_EINTR             |10004|一个封锁操作被对 WSACancelBlockingCall 的调用中断。 |
|SOCKET_EBADF             |10009|提供的文件句柄无效。 |
|SOCKET_EACCES            |10013|以一种访问权限不允许的方式做了一个访问套接字的尝试。 |
|SOCKET_EFAULT            |10014|系统检测到在一个调用中尝试使用指针参数时的无效指针地址。 |
|SOCKET_EINVAL            |10022|提供了一个无效的参数。 |
|SOCKET_EMFILE            |10024|打开的套接字太多。 |
|SOCKET_EWOULDBLOCK       |10035|无法立即完成一个非阻止性套接字操作。 |
|SOCKET_EINPROGRESS       |10036|目前正在执行一个阻止性操作。 |
|SOCKET_EALREADY          |10037|在一个非阻止性套接字上尝试了一个已经在进行的操作。 |
|SOCKET_ENOTSOCK          |10038|在一个非套接字上尝试了一个操作。 |
|SOCKET_EDESTADDRREQ      |10039|请求的地址在一个套接字中从操作中忽略。 |
|SOCKET_EMSGSIZE          |10040|一个在数据报套接字上发送的消息大于内部消息缓冲区或其他一些网络限制，或该用户用于接收数据报的缓冲区比数据报小。 |
|SOCKET_EPROTOTYPE        |10041|在套接字函数调用中指定的一个协议不支持请求的套接字类型的语法。 |
|SOCKET_ENOPROTOOPT       |10042|在 getsockopt 或 setsockopt 调用中指定的一个未知的、无效的或不受支持的选项或层次。 |
|SOCKET_EPROTONOSUPPORT   |10043|请求的协议还没有在系统中配置，或者没有它存在的迹象。 |
|SOCKET_ESOCKTNOSUPPORT   |10044|在这个地址家族中不存在对指定的插槽类型的支持。 |
|SOCKET_EOPNOTSUPP        |10045|参考的对象类型不支持尝试的操作。 |
|SOCKET_EPFNOSUPPORT      |10046|协议家族尚未配置到系统中或没有它的存在迹象。 |
|SOCKET_EAFNOSUPPORT      |10047|使用了与请求的协议不兼容的地址。 |
|SOCKET_EADDRINUSE        |10048|通常每个套接字地址(协议/网络地址/端口)只允许使用一次。 |
|SOCKET_EADDRNOTAVAIL     |10049|在其上下文中，该请求的地址无效。 |
|SOCKET_ENETDOWN          |10050|套接字操作遇到了一个已死的网络。 |
|SOCKET_ENETUNREACH       |10051|向一个无法连接的网络尝试了一个套接字操作。 |
|SOCKET_ENETRESET         |10052|当该操作在进行中，由于保持活动的操作检测到一个故障，该连接中断。 |
|SOCKET_ECONNABORTED      |10053|你的主机中的软件中止了一个已建立的连接。 |
|SOCKET_ECONNRESET        |10054|远程主机强迫关闭了一个现有的连接。 |
|SOCKET_ENOBUFS           |10055|由于系统缓冲区空间不足或队列已满，不能执行套接字上的操作。 |
|SOCKET_EISCONN           |10056|在一个已经连接的套接字上做了一个连接请求。 |
|SOCKET_ENOTCONN          |10057|由于套接字没有连接并且(当使用一个 sendto 调用发送数据报套接字时)没有提供地址，发送或接收数据的请求没有被接受。 |
|SOCKET_ESHUTDOWN         |10058|由于以前的关闭调用，套接字在那个方向已经关闭，发送或接收数据的请求没有被接受。 |
|SOCKET_ETOOMANYREFS      |10059|对某个内核对象的引用过多。 |
|SOCKET_ETIMEDOUT         |10060|由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。 |
|SOCKET_ECONNREFUSED      |10061|由于目标计算机积极拒绝，无法连接。 |
|SOCKET_ELOOP             |10062|无法转换名称。 |
|SOCKET_ENAMETOOLONG      |10063|名称组件或名称太长。 |
|SOCKET_EHOSTDOWN         |10064|由于目标主机坏了，套接字操作失败。 |
|SOCKET_EHOSTUNREACH      |10065|套接字操作尝试一个无法连接的主机。 |
|SOCKET_ENOTEMPTY         |10066|不能删除目录，除非它是空的。 |
|SOCKET_EPROCLIM          |10067|一个 Windows 套接字操作可能在可以同时使用的应用程序数目上有限制。 |
|SOCKET_EUSERS            |10068|限额不足。 |
|SOCKET_EDQUOT            |10069|磁盘限额不足。 |
|SOCKET_ESTALE            |10070|文件句柄引用不再可用。 |
|SOCKET_EREMOTE           |10071|项目在本地不可用。 |
|SOCKET_EDISCON           |10101|由 WSARecv 或 WSARecvFrom 返回表示远程方面已经开始了关闭步骤。 |
|SOCKET_SYSNOTREADY       |10091|因为它使用提供网络服务的系统目前无效，WSAStartup 目前不能正常工作。 |
|SOCKET_VERNOTSUPPORTED   |10092|不支持请求的 Windows 套接字版本。 |
|SOCKET_NOTINITIALISED    |10093|应用程序没有调用 WSAStartup，或者 WSAStartup 失败。 |
|SOCKET_HOST_NOT_FOUND    |11001|不知道这样的主机。 |
|SOCKET_TRY_AGAIN         |11002|这是在主机名解析时通常出现的暂时错误，它意味着本地服务器没有从权威服务器上收到响应。 |
|SOCKET_NO_RECOVERY       |11003|在数据库查找中出现一个不可恢复的错误。 |
|SOCKET_NO_DATA           |11004|请求的名称有效，但是找不到请求的类型的数据。 |
|SOCKET_NO_ADDRESS        |11004|请求的名称有效，但是找不到请求的类型的数据。 |
