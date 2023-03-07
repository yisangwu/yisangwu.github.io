---
layout: post
title: Python多线程，多进程请求
tags: python requests
categories: python  
---

为测试一个单进程服务的接口耗时情况，使用线程、进程同时请求。

```python
# coding=utf8

import requests
import threading

def func_test():
    """
    常规requests请求
    """
    requests.get('http://127.0.0.1:8080/{}'.format('test_0'))
    requests.get('http://127.0.0.1:8080/{}'.format('test_1'))



def func_thread():
    """
    多线程
    """
    class CusThread(threading.Thread):
        def __init__(self, func_name):
            super().__init__()
            self.func_name = func_name.strip()

        def run(self):
            requests.get('http://127.0.0.1:8080/{}'.format(func_name))

    CusThread(func_name='test_0').start()
    CusThread(func_name='test_1').start()



def func_process():
    """
    多进程
    """
    import multiprocessing
    url = ['http://127.0.0.1:8080/test_0', 'http://127.0.0.1:8080/test_1']

    def query_test(key):
        requests.post(url(key), timeout=30)

    if __name__ == "__main__":
        for i in range(len(url)):
            p = multiprocessing.Process(target=query_test, args=(i,))
            p.start()


if __name__ == '__main__':
    func_test()
    func_thread()
    func_process()

```