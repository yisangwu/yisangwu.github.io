---
layout: post
title: CentOS7 安装 FFmpeg
tags: CentOS ffmpeg php
categories: CentOS
---
<style type="text/css">
    p{text-indent: 20px}
</style>

<p>为社区的丰富多样化，社区的文章新增内嵌视频，对视频的处理，除了需要获得视频文件的总长度时间和创建时间，还要视频的第一帧截图。前端需要视频的属性和帧截图，全部由服务器来处理。这样，运营和产品只需要添加标题，上传视频。</p>

### FFmpeg
<p>是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。它包括了目前领先的音/视频编码库libavcodec。 </p>
ffmpeg主要组成部分：
1. libavformat：用于各种音视频封装格式的生成和解析，包括获取解码所需信息以生成解码上下文结构和读取音视频帧等功能，包含demuxers和muxer库；  
2. libavcodec：用于各种类型声音/图像编解码；  
3. libavutil：包含一些公共的工具函数；  
4. libswscale：用于视频场景比例缩放、色彩映射转换；  
5. libpostproc：用于后期效果处理；  
6. ffmpeg：命令行工具，用来对视频文件转换格式，也支持对电视卡实时编码；  
7. ffsever：HTTP多媒体实时广播流服务器，支持时光平移；  
8. ffplay：简单的播放器，使用ffmpeg库解析和解码，通过SDL显示；  

### 官网及下载地址：
- 官网： <http://www.ffmpeg.org/>
- 下载页面：<http://www.ffmpeg.org/download.html>
- 各个版本源码下载地址： <http://ffmpeg.org/releases/>

>下载FFmpeg，尽量下载安装最新版本, 此时版本选择3.2.4（"Hypatia"）。

### 安装步骤：
1. 下载源码：
```shell
$ wget  http://ffmpeg.org/releases/ffmpeg-3.2.4.tar.gz
```
2. 解压：
```shell
$ tar -zxvf ffmpeg-3.2.4.tar.gz
```
3. 进入目录：
```shell
$ cd ffmpeg-3.2.4
```
4. 检查配置环境, 生成 Makefile：
```shell
$ ./configure --enable-shared
```
5. 编译，安装：
```shell
$ make && make  install
```
#### >> 可能出现的问题：
```shell
error: yasm/nasm not found or too old. Use –disable-yasm for a crippled build.
```
下载安装Yasm，再重新编译，安装ffmpeg就可以：
```shell
$ wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
```
解压：
```shell
$ tar -zxvf  yasm-1.3.0.tar.gz
```
进入目录：
```shell
$ cd  yasm-1.3.0
```
检查配置环境, 生成 Makefile：
```shell
$ ./configure
```
编译，安装：
```shell
$ make && make  install
```

6. 继续编译，安装ffmpeg：
```shell
$ make && make  install
```
7. 检查是否安装成功, 执行命令:
```shell
$ ffmpeg
```
***
### PHP使用FFmpeg：

<p>因使用 FFmpeg 是走命令执行的方式，所以，需要调整php.ini配置。修改disable_functions，根据使用的命令行执行函数，去掉相应的函数，exec或system。</p>
修改完保存后, php-fpm要重新加载的配置, 执行命令重载配置：
```shell
$ service php-fpm reload
```
示例代码：
```php
<?php
   /**
     * 获得视频文件的总长度时间和创建时间
     * @param string $file 服务器上的视频文件，绝对路径
     * @return array [] / [总时长，创建时间，视频大小]
     */ 
    function getTime($file){

        $file = trim($file);
        if(empty($file) || !file_exists($file) || !is_file($file)){
            return [];
        }
        // 获取视频信息
        @exec(
            sprintf("ffmpeg -i %s 2>&1 | grep -E 'Duration|Video'", $file), 
            $info 
        );
        if( empty($info) ){
            $vtime = 0;
        }else{
            $info_arr   = explode(",",$info[0]); //Duration: 00:03:53.96
            $vtime      = str_replace('Duration: ', '', trim($info_arr[0]));
            $vtime      = substr($vtime, 0, strpos($vtime, '.'));
            $px_xp_arr  = explode(" ", $info[1]);
            $size       = trim($px_xp_arr[13]);
        }
        // 截图视频帧
        // -ss 开始时间， -t 时长，从00:00:04开始，截取1秒时长的视频，保存为 尺寸640x360名称为frame.jpg的图片
        @exec(
            sprintf("ffmpeg -i %s -y -f mjpeg -ss 00:00:04 -t 00:00:01 -s 640x360 frame.jpg", 
                $file)
        );

        $ctime = date("Y-m-d H:i:s",filectime($file) );//创建时间
        return array('vtime'=>$vtime, 'ctime'=>$ctime, 'size'=>$size );
    }
```
