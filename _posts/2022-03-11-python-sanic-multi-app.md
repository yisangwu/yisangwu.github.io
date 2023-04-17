---
layout: post
title: python框架Sanic多个app服务
tags: python sanic 
categories: python  
---

- 在主进程中使用装饰器main_process_start添加后台任务会导致子进程worker无法启动，不能对外提供服务。
- 在子进程中使用app.add_task添加后台任务，在多个worker时，会被同时多次执行。  

为避免以上的两个问题，尝试拆分为两个独立服务，
* 一个app在主进程添加后台任务，只跑定时任务。
* 一个app跑多个worker，对外提供web服务， 

测试脚本如下：


```shell
# 定时服务，因为在主进程，worker不提供服务
$ sanic t_sanic_final.app_sched --host=0.0.0.0 --port=8000 --workers=1

# web服务，提供http服务, worker为子进程数量
$ sanic t_sanic_final.app_worker --host=0.0.0.0 --port=8001 --workers=2

# 部署：
1. supervisor + saninc
2. supervisor + sanic + gunicon
```


```python
# coding=utf8

import os
import time
import asyncio
from sanic import Sanic, text
from apscheduler.schedulers.blocking import BlockingScheduler

app_worker = Sanic('myapp_worker')
app_sched = Sanic('myapp_sched')

def task_func():
    """
    间隔执行
    """
    print("apscheduler task interval go >>>>>>{}, pid:{}".format(time.time(), os.getpid()))

def task_cron_func():
    """
    crontab执行
    """
    print("apscheduler task cron go >>>>>>{}, pid:{}".format(time.time(), os.getpid()))

@app_sched.main_process_start
async def main_start(*_):
    while True:
        sched = BlockingScheduler()
        sched.add_job(task_func, 'interval', seconds=3, id='job_sec_3')  # 间隔执行 
        sched.add_job(task_cron_func, 'cron', minute='*/1')  #  crontab 执行
        await sched.start()

@app_worker.get("/")
async def foo_handler(request):
    return text("foo_handler foo_handler!")

```

### 输出：
```shell

// 8000 定时服务：

[2023-04-17 13:42:22 +0000] [268] [INFO] Goin' Fast @ http://0.0.0.0:8000
apscheduler task interval go >>>>>>1681739574.3421297, pid:320
apscheduler task interval go >>>>>>1681739577.3421874, pid:320
apscheduler task cron go >>>>>>1681739580.0014553, pid:320

// 80001 web服务：

[2023-04-17 13:43:26 +0000] [322] [INFO] Goin' Fast @ http://0.0.0.0:8001
[2023-04-17 13:43:26 +0000] [323] [INFO] Starting worker [323]
[2023-04-17 13:43:26 +0000] [324] [INFO] Starting worker [324]

$ curl -i http://127.0.0.1:8001
HTTP/1.1 200 OK
content-length: 24
connection: keep-alive
content-type: text/plain; charset=utf-8

foo_handler foo_handler!sh-4.4# 
sh-4.4# curl -i http://127.0.0.1:8001
HTTP/1.1 200 OK
content-length: 24
connection: keep-alive
content-type: text/plain; charset=utf-8

foo_handler foo_handler!
```
