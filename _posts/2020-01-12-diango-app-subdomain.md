---
layout: post
title: Django 每个App配置独立域名
tags: python django
categories: python django
---

<style type="text/css">
    dd{text-indent: 20px}
</style>

> 环境：Python 3.8.1 + Django 2.2.12

为整合流程规范，部署了综合型的项目，包含了 task (需求系统)，doc (文档系统)，sso (单点登录系统)，

大致的目录结构：
```shell
CellMiddle  -- 项目根目录
├─task --- 需求系统
│  ├─migrations
│  ├─static
│  └─templates
├─logs --- 日志目录
├─doc --- 文档系统
│  ├─data
│  ├─migrations
│  ├─static
│  └─templates
├─sso --- 单点登录，权限控制系统
│  ├─migrations
│  ├─static
│  └─templates
├─upload ---文件资源上传目录
├─static --- 静态资源
├─templates --- 公共模板
└─CellMiddle --- 项目主目录
│    ├─config --- 公共配置
│    ├─helper --- 公共辅助类，ex：时间，日期，字符串，响应等
│    └─loader --- 第三方库的实例化加载，ex：Redis，Memcached等
│    │  settings.py
│    │  urls.py
│    │  wsgi.py
│    │  __init__.py
├─manage.py
└─envConf  -- 部署配置文件：requirement.txt， host.conf, supervisor, gunicorn 等
```

使用 nginx + gunicorn + supervisor 部署，使用域名 cell.hao456.com 解析指向服务器，
nginx 监听 gunicorn 绑定的端口，可以正常访问：
```shell
http://cell.hao456.com/task
http://cell.hao456.com/doc
http://cell.hao456.com/sso
```
虽然同一个域名路由控制来访问，也是可以的。但还是想像PHP一样只要独立入口文件，每个app独立使用不同子域名，如：
```shell
http://task.hao456.com 指向task
http://doc.hao456.com  指向doc
http://sso.hao456.com  指向sso
```

尝试了几种办法：
1. ##### nginx 层转发；
    - 修改proxy指向，location 要加app，static 等。
2. ##### 每个app独立一个 wsgi.py 文件，使用gunicorn单独起服务；
    - 增加 task_wsgi.py, doc_wsgi.py, sso_wsgi.py，使用 supervisor起三个gunicorn服务。
3. ##### 使用django-hosts；
    - 添加MIDDLEWARE中间件，实现host与urls的namespace关联。nginx的conf配置不用改动，只需要将
所有子域名的监听，一起代理指向gunicorn端口，

```shell
$ cat hosts.conf
server {
    listen 80;
    server_name task.hao456.com doc.hao456.com sso.hao456.com;
    location / {
        proxy_pass http://127.0.0.1:10888;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
    }
    ... ...
}
```

### django-hosts实现：
1. pip 安装 django-hosts， 这里用到的是 4.0版本：
```shell
    $ pip install django-hosts==4.0  # root权限
```
2. 修改settings.py：
    - 将django-hosts添加到app：
```python
    INSTALLED_APPS = [
        ...
        'django_hosts',
        'sso',
        'doc',
        'task',
    ]
```
    - 添加Django的hosts配置：
```python
    ROOT_HOSTCONF = 'CellMiddle.hosts'  # host配置
    DEFAULT_HOST = 'sso'  # 默认的域名
```    

    - 将django_hosts添加到中间件MIDDLEWARE：
```python
    MIDDLEWARE = [
    'django_hosts.middleware.HostsRequestMiddleware',  # 首要加
    ...
    'django_hosts.middleware.HostsRequestMiddleware',  # 尾要加
]
```

3. 在项目主目录下settings.py同级，添加 hosts.py（注意和 settings 里面配置的ROOT_HOSTCONF一致）：
```shell
...
└─CellMiddle
    │  hosts.py  --- 新增文件
    │  settings.py
    │  urls.py
    │  wsgi.py
    │  __init__.py
```
内容为：  
```python
# coding=utf8
    """
    django-hosts
    """
    from django.conf import settings
    from django_hosts import patterns, host

    host_patterns = patterns(
        '',
        host('sso', settings.ROOT_URLCONF, name='sso'),
        host('doc', 'doc.urls', name='doc'),
        host('task', 'task.urls', name='task'),
    )
``` 

4. 路由配置urls.py，添加路由对应的appname（app名称）, namespace(域名)：
```python
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include(('sso.urls', 'sso'), namespace='sso')),  # 默认sso
        path('doc', include(('doc.urls', 'doc'), namespace='doc')),  # 文档
        path('task', include(('task.urls', 'task'), namespace='task')),  # 需求
    ]
```

5. reload 重载服务：
  - DNS 确认子域名的解析;
  - 检查nginx的配置有效性，重载nginx;
  - 动态更新Web服务 或 在 supervisorctl重启Web服务；  
```shell
$ kill -HUP PID  # root权限
$ supervisorctl restart program_name服务名  # root权限
```

6. 检查子域名的访问，app对应的路由，静态资源访问等是否正常；
