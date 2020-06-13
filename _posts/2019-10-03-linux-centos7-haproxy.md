<!--
 * @Author: your name
 * @Date: 2020-06-13 14:44:52
 * @LastEditTime: 2020-06-13 17:49:53
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \dramaclub_backendd:\wamp64-3.2.0\www\mix_url\x.md
--> 

跨区域部署业务时，为最小化部署，使用正向代理将请求转发到中心服务器，转发请求包括http 和 tcp。当然Haproxy也是可以做负载均衡的，尤其是在多TCP后端服务器部署下。


### HAProxy 
是一款提供高可用性、负载均衡以及基于TCP（第四层）和HTTP（第七层）应用的代理软件，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。

服务器环境：
> CentOS Linux release 7.8

安装方式，yum安装，Root权限。
> > 注意CentOS 8安装时会不一样

1. 查找yum源中的haproxy：
```shell
# yum list | grep haproxy
haproxy.x86_64                            1.5.18-9.el7                   @base  
pcp-pmda-haproxy.x86_64                   4.3.2-7.el7_8                  updates
```

2. 创建日志目录：
```shell
# mkdir /var/log/haproxy
# chmod a+w /var/log/haproxy
```

3. 开启rsyslog记录haproxy日志：
```shell
# vim /etc/rsyslog.conf

# Provides UDP syslog reception
$ModLoad imudp    # 
$UDPServerRun 514

# haproxy log
local0.*    /var/log/haproxy/haproxy.log  # 添加
```

3. 修改 /etc/sysconfig/rsyslog 文件：
```shell
# Options for rsyslogd
# Syslogd options are deprecated since rsyslog v3.
# If you want to use them, switch to compatibility mode 2 by "-c 2"
# See rsyslogd(8) for more details
SYSLOGD_OPTIONS="-r -m 0 -c 2"
```
4. 重启rsyslog，使日志配置生效：
```shell
# systemctl restart rsyslog
```

5. 修改haproxy的配置文件，/etc/haproxy/haproxy.cfg：
```shell
###########全局配置#########
    global
        log 127.0.0.1 local0 err  # 日志类型，为不影响性能使用err
        daemon
        #nbproc 1     #进程数量 
        maxconn 4096  #最大连接数 
        #user haproxy  #运行用户  
        #group haproxy #运行组 
        chroot      /var/lib/haproxy
        pidfile     /var/run/haproxy.pid
 
########默认配置############
    defaults
        log global
        mode http            #默认模式{ tcp|http|health }
        option httplog       #日志类别,采用httplog
        option dontlognull   #不记录健康检查日志信息  
        retries 2            #3次连接失败就认为服务器不可用
        option forwardfor    except 127.0.0.0/8  #后端服务获得真实ip,在HTTP请求中添加"HTTP_X_FORWARDED_FOR"字段
        option httpclose     #请求完毕后主动关闭http通道
        option abortonclose  #服务器负载很高，自动结束比较久的链接  
        maxconn 10000        #最大连接数  
        timeout connect 5m   #连接超时   m(分钟)
        timeout client 1m    #客户端超时  
        timeout server 1m    #服务器超时  
        timeout check 10s    #心跳检测超时  s(秒)
        balance leastconn    #负载均衡方式，最少连接 

########统计页面配置############
    listen stats                 
        bind 0.0.0.0:1080        #监听端口，云服务器要开公网端口
        mode http                #http的7层模式
        option httplog
        log 127.0.0.1 local0 err #错误日志记录
        stats refresh 15s        #每隔15秒自动刷新监控页面
        maxconn 10               #最大连接数，同时访问stats页面的个数
        stats uri /status        #状态页面 http//ip:1080/status
        stats realm Haproxy\ Statistics
        stats auth admin:admin   #用户和密码
        stats hide-version       #隐藏版本信息  
        stats admin if TRUE      #设置手工启动/禁用

########frontend前端配置############## 
    frontend http_local_60_frontend
        bind *:8577 
        mode http
        default_backend http_local_60_backend

########backend后端配置##############
    backend http_local_60_backend
        mode http
        server http_80 192.168.101.60:80 
        
######## frontend 和 backend 写在一起##############  
    listen tcp-frontend
        bind *:8577  # 监听端口，要开公网访问
        balance roundrobin  # 基于权重轮询，动态算法
        mode tcp
        option tcplog
        server tcp-backend 192.168.101.60:8577
        server tcp-backend 192.168.101.61:8577 check # 对当前server做健康状态检测
        server tcp-backend 192.168.101.62:8577 check backup  # backup 设定当前server为备用服务器
```

6. 检查haproxy配置是否有效：
```shell
# haproxy -c -f /etc/haproxy/haproxy.cfg 
...
Configuration file is valid  # 有效，警告可以处理，一般都是 log类型，option tcplog, http的option forwardfor
```

7. 启动haproxy：
```shell
$ systemctl start haproxy.service
```

8. 查看监听端口，监听端口为bind的端口：
```shell
# netstat -lnpt
```

9. haproxy管理命令：
```shell
# 启动
$ systemctl start haproxy.service
# 停止
$ systemctl stop haproxy.service
# 修改配置重新加载
$ systemctl reload haproxy.service
# 重启
$ systemctl restart haproxy.service
```
