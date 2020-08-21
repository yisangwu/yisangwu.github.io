---
layout: post
title: php socket数据发送接收MSG常量
tags: php, socket
categories: php
---
>php版本：
>>PHP 7.4.0 (cli) (built: Nov 27 2019 10:14:18) ( ZTS Visual C++ 2017 x64 )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Xdebug v2.9.2, Copyright (c) 2002-2020, by Derick Rethans

|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |

## windows下：
| 常量 | 数值 | 说明 | socket_recv | socket_send |
| :------- | ------- | :------- | ------- | ------- |
| MSG_OOB        | 1 | 处理超出边界的数据。 | √ | √ |
| MSG_PEEK       | 2 | 从接受队列的起始位置接收数据，但不将他们从接受队列中移除。 | √ | - |
| MSG_WAITALL    | 8 | 在接收到至少 len 字节的数据之前，造成一个阻塞，并暂停脚本运行（block）。但是， 如果接收到中断信号，或远程服务器断开连接，该函数将返回少于 len 字节的数据。 | √ | - |
| MSG_CTRUNC     | 512 | 控制数据被截断 | - | - |
| MSG_TRUNC      | 256 | 返回数据包实际长度即使被截断 | - | - |
| MSG_DONTROUTE  | 4 | 勿将数据包路由出本地网络 | - | - |
| MSG_ERRQUEUE   | 4096 | 接受错误信息作为辅助数据 | - | - |

## Linux下：
| 常量 | 数值 | 说明 | socket_recv | socket_send |
| :------- | ------- | :------- | ------- | ------- |
|MSG_OOB        |1|处理超出边界的数据。接受带外数据。|√|√|
|MSG_PEEK       |2|从接受队列的起始位置接收数据，但不将他们从接受队列中移除。|√|-|
|MSG_WAITALL    |256|在接收到至少 len 字节的数据之前，造成一个阻塞，并暂停脚本运行（block）。但是， 如果接收到中断信号，或远程服务器断开连接，该函数将返回少于 len 字节的数据。|√|-|
|MSG_DONTWAIT   |64|如果制定了该flag，函数将不会造成阻塞，即使在全局设置中指定了阻塞设置。|√|-|
|MSG_EOR        |128|接收记录结束符，在 Windows 平台上无效。|-|√|
|MSG_EOF        |512|标记记录结束，在 Windows 平台上无效。|-|√|
|MSG_DONTROUTE  |4|勿将数据包路由出本地网络|-|√|
|MSG_CTRUNC     |8|控制数据被截断|-|-|
|MSG_TRUNC      |32|返回数据包实际长度即使被截断|-|-|
|MSG_CONFIRM    |2048|提供链路层反馈以保持地址映射有效|-|-|
|MSG_ERRQUEUE   |8192|接受错误信息作为辅助数据|-|-|
|MSG_NOSIGNAL   |16384|在无连接的套接字不产生信号SIGPIPE|-|-|
|MSG_MORE       |32768|允许延迟并写更多数据|-|-|
|MSG_WAITFORONE |65536|-|-|-|
