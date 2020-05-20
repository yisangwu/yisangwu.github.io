---
layout: post
title: python 3.7 crash after install readline
tags: python, readline
categories: python
---


> pip install readline 安装了readline模块之后，python控制台崩溃：

执行结果，如下：
```shell
>>> 1+1 
*** Error in `python': free(): invalid pointer: 0x00007f13bf7a7af0 ***
======= Backtrace: ========= 
/usr/lib64/libc.so.6(+0x81679)[0x7f13be21a679] 
/usr/local/lib/libpython3.7m.so.1.0(PyOS_Readline+0x165)[0x7f13bf12ba3a] 
/usr/local/lib/libpython3.7m.so.1.0(+0xab14f)[0x7f13bf16e14f] 
/usr/local/lib/libpython3.7m.so.1.0(PyTokenizer_Get+0x220)[0x7f13bf16f2a0] 
/usr/local/lib/libpython3.7m.so.1.0(+0xaa413)[0x7f13bf16d413]
/usr/local/lib/libpython3.7m.so.1.0(PyParser_ASTFromFileObject+0x74)[0x7f13bf14fbe3] 
/usr/local/lib/libpython3.7m.so.1.0(+0x8d3a4)[0x7f13bf1503a4] 
/usr/local/lib/libpython3.7m.so.1.0(PyRun_InteractiveLoopFlags+0xd8)[0x7f13bf1505c9]
/usr/local/lib/libpython3.7m.so.1.0(PyRun_AnyFileExFlags+0x39)[0x7f13bf15067d] 
/usr/local/lib/libpython3.7m.so.1.0(+0x1f4f3b)[0x7f13bf2b7f3b] 
/usr/local/lib/libpython3.7m.so.1.0(_Py_UnixMain+0x2c)[0x7f13bf2b823c] 
/usr/lib64/libc.so.6(__libc_start_main+0xf5)[0x7f13be1bb505] 
python[0x40069e] 
======= Memory map: ======== 
00400000-00401000 r-xp 00000000 fd:00 764448 /usr/local/bin/python3.7 
00600000-00601000 r--p 00000000 fd:00 764448 /usr/local/bin/python3.7 
00601000-00602000 rw-p 00001000 fd:00 764448 /usr/local/bin/python3.7 
01467000-0150d000 rw-p 00000000 00:00 0 `
```
### 解决：
**安装gnureadline而不是readline**
```shell
$ pip uninstall readline 
$ pip install gnureadline
```
