---
layout: post
title: php安装扩展imagick
tags: php extension ImageMagick imagick
categories: php 
---
<style type="text/css">
    p{text-indent: 20px}
</style>
<p>之前使用GD库可以满足项目中需要的头像，图片的处理。包括群和讨论组的类似QQ讨论组九宫格头像拼接也用GD库实现了。  </p>
<p>但是，在用户量逐步涨上来的时候，效率上就满足不了。即便做了异步队列来异步生成讨论的头像，还是会有延迟显示的问题出现。  </p>
<p>索性，直接换上<span style="color:red">imagick。</span></p>

<p>先解释下名词，这两者的关系；</p>
1. ImageMagick:
>ImageMagick® 是用来创建，编辑，合并位图图像的一套组件。 它能够用于读取，转换，写入多种不同格式的图像。 包含 DPX, EXR, GIF, JPEG, JPEG-2000, PDF, PhotoCD, PNG, Postscript, SVG, 和 TIFF。
2. imagick：
>imagick 是用 ImageMagic API 来创建和修改图像的PHP官方扩展。
>imagick的PHP扩展库地址：http://pecl.php.net/package/imagick


### 一、安装<span style="color:red">ImageMagick</span>：

两种方式安装：
- 源码安装;
- rpm软件包安装;
>推荐使用rpm软件包安装，注意root权限同时安装 ImageMagick 和 ImageMagick-libs。

这里使用的是源码安装，注意是root权限，步骤如下：

1.下载源码：  
```shell
$ wget http://www.imagemagick.org/download/ImageMagick.tar.gz
```
2.解压：  
```shell
$ tar zxf ImageMagick.tar.gz
```
3.进入目录：  
```shell
$ cd ImageMagick-7.0.7-15
```
4.检查配置环境, 生成 Makefile：  
```shell
$ ./configure  --prefix=/usr/local/ImageMagick-7
```
官方推荐高级用户使用：
```shell
$ ./configure --with-modules --enable-shared --with-perl
```
5.编译并安装：  
```shell
$ make && make install
```
6.检查是否安装成功：
```shell
$ convert -version
```

### 二、安装PHP扩展<span style="color:red">imagick</span>：
也有两种方式安装：
- 源码安装;
> 需要指定 ImageMagick 的安装目录
- pecl安装;
> rpm软件包安装 ImageMagick ，使用 pecl 安装更快捷。  
> root 权限执行: $ pecl install imagick 

      下载：wget  http://pecl.php.net/get/imagick-3.4.3.tgz  

这里也使用源码安装imagick扩展，步骤如下：
1. 下载imagick：
```shell
$ wget http://pecl.php.net/get/imagick-3.4.3.tgz
```
2. 解压：
```shell
$ tar zxvf imagick-3.4.3.tgz
```
3. 进入目录：
```shell
$ cd imagick-3.4.3
```
4. 生成configure配置文件：
```shell
$ phpize
```
5. 检查配置环境, 生成 Makefile, 指定 ImageMagick 路径：  
```shell
$ ./configure --with-php-config=/usr/local/php/bin/php-config  \
--with-imagick=/usr/local/ImageMagick-7
```
6. 编译并安装：  
```shell
$ make && make install
```
>备注：在安装过程中出现错误，一般是由于缺少编译工具包导致，可根据提示参照第一步安装相应的工具包即可。安装完成之后，出现下面的界面：
```shell
Installing  shared  extensions: /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/
Installing  header  files:  /usr/local/php/include/php/
```
7.  配置 php 支持 imagick：
    - 查看php.ini路径：
```shell
    $ php --ini
    Configuration  File  (php.ini)  Path:  /usr/local/php/etc
    Loaded  Configuration  File:  /usr/local/php/etc/php.ini
    Scan  for  additional  .ini  files  in:  /usr/local/php/conf.d
    Additional  .ini  files  parsed:  /usr/local/php/conf.d/002-zendguardloader.ini
```
    - 编辑配置文件，添加扩展支持：
```shell
$ vim /usr/local/php/etc/php.ini
```
在最后一行添加, extension="imagick.so"

8. 重载php-fpm:
```shell
$ service php-fpm reload
```
9. 检查扩展是否安装成功：
```shell
$ php -m | grep imagick
```
10. 查看imagick扩展信息：
```shell
$ php --ri imagick
```
