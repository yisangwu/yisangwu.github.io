---
layout: post
title: python函数作为参数动态导入
tags: python,pickle
categories: python
---


最近封装一个类，需要动态导入包函数，作为参数，传参给下游方法调用。想到flask的blueprint，还有pickle包，两种不同的实现方式，尝试了下。


### pickle序列化函数
需要传参前，导入方法，再序列化方法为
#### 1. 项目目录结构
```shell
_modTest
 | __init__.py
 | hello_world.py
```

- obj使用时：

```python
# coding=utf8

import pickle
from modTest.hello_world import WorldClass

func = WorldClass().real_func
# 传参时，传递func_dumps
# func_dumps = pickle.dumps(func, 1)  0/1都是可以的
func_dumps = pickle.dumps(func, 0)

# 使用时
func_loads = pickle.loads(func_dumps)
print(func_loads([1,2,3]))
```

 - 字符串传参时：

```python
# coding=utf8

import pickle
from modTest.hello_world import WorldClass

func = WorldClass().real_func
# 传参时，传递func_dumps
# func_dumps = pickle.dumps(func, 1)  0/1都是可以的
func_dumps = pickle.dumps(func, 0).decode('utf-8')

# 使用时
func_loads = pickle.loads(func_dumps.encode('utf-8'))
print(func_loads([1,2,3]))
```

### 内置函数__import__
多层次包结构，都可以使用。

```python
def import_exec_func(mod_str, func_name, *args, **kwargs):
    """
    动态导入包的类方法
    :param mod_str 包名.文件名.类名
    :param func_name 类的方法名
    :param args
    :param kwargs
    :return
    """
    mod_class_str, _sep, class_str = mod_str.rpartition('.')
    tuple_import = __import__(mod_class_str)
    if not tuple_import:
        raise ImportError("not found mod_class:{}".format(mod_class_str))

    class_obj = getattr(sys.modules[mod_class_str], class_str)
    if not class_obj:
        raise ImportError("not found class:{}".format(class_str))

    func = getattr(class_obj, func_name)
    if not func:
        raise ImportError("not found function:{}".format(func_name))

    return func(args, kwargs)


# 执行调用
print(import_exec_func("modTest.hello_world.WorldClass", "real_func", 4,5,6, a="aa1122"))
```
