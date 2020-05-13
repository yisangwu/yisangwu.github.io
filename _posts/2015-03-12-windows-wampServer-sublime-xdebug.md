---
layout: post
title: windows下wampServer配置sublime的Xdebug环境
tags: php, xdebug
categories: php
---
<style type="text/css">
    p{text-indent: 20px}
</style>
> 环境：windows 64位 + wampServer 64位 + sublime Text

### Xdebug 
<p>Xdebug是一个开放源代码的php扩展，使用DBGp协议，用于进行跟踪,调试和PHP程序性能分析。</p>

### 下载地址：
- Xdebug官网： <https://xdebug.org/>
- sublime text 的 xdebug扩展：
https://packagecontrol.io/packages/Xdebug%20Client


下载对应PHP版本的xdebug扩展，windows下使用dll后缀
### 1. wampServer 修改配置，添加Xdebug扩展：
  修改 F:/wamp64/bin/php/php5.6/phpForApache.ini
 添加xdebug扩展配置，重启wampserver服务；
> 注意：
>>修改的是apache使用的php配置  
>>使用的是zend_extension
```shell
[xdebug]
zend_extension="F:/wamp64/bin/php/php5.6/zend_ext/php_xdebug.dll"
xdebug.remote_enable = on
xdebug.profiler_enable = on
xdebug.profiler_enable_trigger = on
xdebug.profiler_output_name = cachegrind.out.%t.%p
xdebug.profiler_output_dir ="F:/wamp64/tmp"
xdebug.show_local_vars=0
xdebug.remote_host = "127.0.0.1"
xdebug.remote_port = 9966   # 端口自定义
xdebug.remote_handler = "dbgp"
xdebug.remote_mode = req
xdebug.remote_connect_back = 1
xdebug.remote_log="F:/wamp64/tmp/xdebug.log"  //查看xdebug日志，成功断点日志是串xml
```

### 2. sublime 安装 xdebug-client插件：
用户设置：
```shell
{
    "super_globals": true,
    "close_on_stop": true,
    "host": "127.0.0.1",
    "port": 9966,
    "ide_key": "xdebug", // sublime的默认ide_key=sublime.xdebug
    "python_path" : "D:/Python",  # 配置python，为排查日志使用
}
```

比较容易忽略的操作：  
在sublime 项目目录，www下执行菜单： 项目-->项目另存为， 保存为 xdebug.sublime-project：
```shell
{
	"folders":
	[
		{
			"path": "."
		}
	],
    "settings":
    {
        "xdebug":
        {
            "close_on_stop": true,     //添加
            "super_globals": true,     //添加
            "url": "http://127.0.0.1"  //添加
        }
    }
}
```

重启sublime，打开:视图-> 控制台，查看有无错误。

#### 3. Xdebug 使用：

1，在sublime 菜单栏，tools -> Xdebug -> Start Debugging， sublime界面会出现xdebug的面板，有四个tab。

2,  在浏览器访问页面，url后面带参数：XDEBUG_SESSION_START=xdebug (sublime定义的id_key)
> 使用浏览器插件，就不用在url带参数，火狐浏览器，about:addons，  使用插件 xdebug-ext，选项中设置ide key = xdebug（sublime设置的key）激活扩展，直接访问URL，和不使用扩展，url带参数XDEBUG_SESSION_START=xdebug，是一样的。

3. 配置xdebug成功时，Xdebug context 显示的是访问页面的数据，变量，server，cookie，session等，点击对应变量，会展开祥情。  
例如：
```shell
$_COOKIE = array[1]
$_ENV = array[0]
$_FILES = array[0]
$_GET = array[0]
$_POST = array[0]
$_REQUEST = array[0]
$_SERVER = array[37]
$arr_letter = array[26]
$arr_num = array[10]
```

#### 4. Xdebug配置错误日志排查：

xdebug.remote_log 日志中的l排查：
   1.   E: Time-out connecting to client (Waited: 200 ms)：  
        解决：  
        查看是不是防火墙开了，如果没有开防火墙，在sublime中restart 下xdebug，浏览器重新请求；
   2.  日志中出现了xml格式的数据，但是sublime界面，Xdebug context 一直是空的：  
       解决：  
       在sublime控制台看到有报错，python解析xml错误，对应的脚本是protocol.py；  

      找到：C:\Users\tencent\AppData\Roaming\Sublime Text\Installed Packages  
       使用360 压缩直接打开：Xdebug Client.sublime-package
       编辑文件：Xdebug Client.sublime-package\xdebug\protocol.py
```python
 # Named entity
    else:
        try:
            # Following are not needed to be converted for XML
            if text[1:-1] == 'amp' or text[1:-1] == 'gt' or text[1:-1] == 'lt':
                pass
            elif text[1:-1] == 'quot':  # 新增
                text = "'"    # 新增
            else:
                text = H.unicode_chr(name2codepoint[text[1:-1]])
        except KeyError:
            pass
    return text
return re.sub('&#?\w+;', convert, string)
```
重新打开在sublime 菜单，tools-> Xdebug -> Start Xdebug Debugging, 启动Xdebug服务。 访问查看Xdebug是否正常。
