---
layout: post
title: Linux下 PHP socket 错误码预定义常量， 对应数值，错误信息
tags: php, socket
categories: php
---

##  Linux下 PHP socket 错误码预定义常量， 对应数值，错误信息！

>php版本：
>>PHP 7.4.0 (cli) (built: Nov 27 2019 10:14:18) ( ZTS Visual C++ 2017 x64 )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Xdebug v2.9.2, Copyright (c) 2002-2020, by Derick Rethans

|错误码预定义常量|  数值| 错误信息|
| :-----| :-----| :-----|
|SOCKET_EPERM             |1|Operation not permitted |
|SOCKET_ENOENT            |2|No such file or directory |
|SOCKET_EINTR             |4|Interrupted system call |
|SOCKET_EIO               |5|Input/output error |
|SOCKET_ENXIO             |6|No such device or address |
|SOCKET_E2BIG             |7|Argument list too long |
|SOCKET_EBADF             |9|Bad file descriptor |
|SOCKET_EAGAIN            |11|Resource temporarily unavailable |
|SOCKET_ENOMEM            |12|Cannot allocate memory |
|SOCKET_EACCES            |13|Permission denied |
|SOCKET_EFAULT            |14|Bad address |
|SOCKET_ENOTBLK           |15|Block device required |
|SOCKET_EBUSY             |16|Device or resource busy |
|SOCKET_EEXIST            |17|File exists |
|SOCKET_EXDEV             |18|Invalid cross-device link |
|SOCKET_ENODEV            |19|No such device |
|SOCKET_ENOTDIR           |20|Not a directory |
|SOCKET_EISDIR            |21|Is a directory |
|SOCKET_EINVAL            |22|Invalid argument |
|SOCKET_ENFILE            |23|Too many open files in system |
|SOCKET_EMFILE            |24|Too many open files |
|SOCKET_ENOTTY            |25|Inappropriate ioctl for device |
|SOCKET_ENOSPC            |28|No space left on device |
|SOCKET_ESPIPE            |29|Illegal seek |
|SOCKET_EROFS             |30|Read-only file system |
|SOCKET_EMLINK            |31|Too many links |
|SOCKET_EPIPE             |32|Broken pipe |
|SOCKET_ENAMETOOLONG      |36|File name too long |
|SOCKET_ENOLCK            |37|No locks available |
|SOCKET_ENOSYS            |38|Function not implemented |
|SOCKET_ENOTEMPTY         |39|Directory not empty |
|SOCKET_ELOOP             |40|Too many levels of symbolic links |
|SOCKET_EWOULDBLOCK       |11|Resource temporarily unavailable |
|SOCKET_ENOMSG            |42|No message of desired type |
|SOCKET_EIDRM             |43|Identifier removed |
|SOCKET_ECHRNG            |44|Channel number out of range |
|SOCKET_EL2NSYNC          |45|Level 2 not synchronized |
|SOCKET_EL3HLT            |46|Level 3 halted |
|SOCKET_EL3RST            |47|Level 3 reset |
|SOCKET_ELNRNG            |48|Link number out of range |
|SOCKET_EUNATCH           |49|Protocol driver not attached |
|SOCKET_ENOCSI            |50|No CSI structure available |
|SOCKET_EL2HLT            |51|Level 2 halted |
|SOCKET_EBADE             |52|Invalid exchange |
|SOCKET_EBADR             |53|Invalid request descriptor |
|SOCKET_EXFULL            |54|Exchange full |
|SOCKET_ENOANO            |55|No anode |
|SOCKET_EBADRQC           |56|Invalid request code |
|SOCKET_EBADSLT           |57|Invalid slot |
|SOCKET_ENOSTR            |60|Device not a stream |
|SOCKET_ENODATA           |61|No data available |
|SOCKET_ETIME             |62|Timer expired |
|SOCKET_ENOSR             |63|Out of streams resources |
|SOCKET_ENONET            |64|Machine is not on the network |
|SOCKET_EREMOTE           |66|Object is remote |
|SOCKET_ENOLINK           |67|Link has been severed |
|SOCKET_EADV              |68|Advertise error |
|SOCKET_ESRMNT            |69|Srmount error |
|SOCKET_ECOMM             |70|Communication error on send |
|SOCKET_EPROTO            |71|Protocol error |
|SOCKET_EMULTIHOP         |72|Multihop attempted |
|SOCKET_EBADMSG           |74|Bad message |
|SOCKET_ENOTUNIQ          |76|Name not unique on network |
|SOCKET_EBADFD            |77|File descriptor in bad state |
|SOCKET_EREMCHG           |78|Remote address changed |
|SOCKET_ERESTART          |85|Interrupted system call should be restarted |
|SOCKET_ESTRPIPE          |86|Streams pipe error |
|SOCKET_EUSERS            |87|Too many users |
|SOCKET_ENOTSOCK          |88|Socket operation on non-socket |
|SOCKET_EDESTADDRREQ      |89|Destination address required |
|SOCKET_EMSGSIZE          |90|Message too long |
|SOCKET_EPROTOTYPE        |91|Protocol wrong type for socket |
|SOCKET_ENOPROTOOPT       |92|Protocol not available |
|SOCKET_EPROTONOSUPPORT   |93|Protocol not supported |
|SOCKET_ESOCKTNOSUPPORT   |94|Socket type not supported |
|SOCKET_EOPNOTSUPP        |95|Operation not supported |
|SOCKET_EPFNOSUPPORT      |96|Protocol family not supported |
|SOCKET_EAFNOSUPPORT      |97|Address family not supported by protocol |
|SOCKET_EADDRINUSE        |98|Address already in use |
|SOCKET_EADDRNOTAVAIL     |99|Cannot assign requested address |
|SOCKET_ENETDOWN          |100|Network is down |
|SOCKET_ENETUNREACH       |101|Network is unreachable |
|SOCKET_ENETRESET         |102|Network dropped connection on reset |
|SOCKET_ECONNABORTED      |103|Software caused connection abort |
|SOCKET_ECONNRESET        |104|Connection reset by peer |
|SOCKET_ENOBUFS           |105|No buffer space available |
|SOCKET_EISCONN           |106|Transport endpoint is already connected |
|SOCKET_ENOTCONN          |107|Transport endpoint is not connected |
|SOCKET_ESHUTDOWN         |108|Cannot send after transport endpoint shutdown |
|SOCKET_ETOOMANYREFS      |109|Too many references: cannot splice |
|SOCKET_ETIMEDOUT         |110|Connection timed out |
|SOCKET_ECONNREFUSED      |111|Connection refused |
|SOCKET_EHOSTDOWN         |112|Host is down |
|SOCKET_EHOSTUNREACH      |113|No route to host |
|SOCKET_EALREADY          |114|Operation already in progress |
|SOCKET_EINPROGRESS       |115|Operation now in progress |
|SOCKET_EISNAM            |120|Is a named type file |
|SOCKET_EREMOTEIO         |121|Remote I/O error |
|SOCKET_EDQUOT            |122|Disk quota exceeded |
|SOCKET_ENOMEDIUM         |123|No medium found |
|SOCKET_EMEDIUMTYPE       |124|Wrong medium type |
