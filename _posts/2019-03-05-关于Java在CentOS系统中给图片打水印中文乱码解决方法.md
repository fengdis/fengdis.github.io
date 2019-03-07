---
layout:     post
title:      关于Java在CentOS系统中给图片打水印中文乱码解决方法
subtitle:   关于Java在CentOS系统中给图片打水印中文乱码解决方法
date:       2019-03-05
author:     fengdi
header-img: img/post-bg-others7.jpg
catalog: true
tags:
    - 中文乱码
    - 图片水印
---

>系统中上传图片为保证版权许给图片打水印，但在linux系统中中文出现乱码，需要给linux安装中文字体

### 1、查看系统字体
在开始安装之前，我们先查看系统中已经安装的字体。要查看系统中已经安装的字体，我们可以使用fc-list命令进行查看。如果系统中没有该命令的话，我们需要先安装相关的软件包。

### 2、安装字体
#### 2.1、安装默认字体
```$xslt
yum install -y fontconfig mkfontscale
```
安装完毕后查看字体安装情况
```$xslt
cat /etc/issue

fc-list
```
如果要查看系统中已经安装的中文字体，我们可以使用如下命令：
```$xslt
fc-list :lang=zh
```
#### 2.1、安装中文字体
系统中没有微软雅黑字体。我们现在需要把MSYH.TTF和MSYHBD.TTF文件上传到linux服务器/usr/share/fonts/目录下。

然后建立字体索引信息，更新字体缓存，使用如下命令：
```$xslt
cd /usr/share/fonts/

mkfontscale

mkfontdir

fc-cache
```
至此，字体已经安装完毕。查看微软雅黑字体，是否安装成功，使用如下命令：
```$xslt
fc-list :lang=zh
```

