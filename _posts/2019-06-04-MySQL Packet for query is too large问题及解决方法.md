---
layout:     post
title:      MySQL Packet for query is too large问题及解决方法
subtitle:   MySQL Packet for query is too large问题及解决方法
date:       2019-06-04
author:     fengdi
header-img: img/post-bg-alibaba.jpg
catalog: true
tags:
    - mysql
    - Packet
---

>项目运行过程中出现数据库max_allowed_packet参数错误，现将整个解决过程记录下来

### 1、问题描述与分析
- Caused by: com.mysql.jdbc.PacketTooBigException: Packet for query is too large (1354 > 1024). You can change this value on the server   
by setting the max_allowed_packet' variable
- MySQL根据配置文件会限制Server接受的数据包大小（默认1024，即1）。有时候插入、更新或查询时数据包的大小，会受 max_allowed_packet 参数限制，导致操作失败。针对该问题，需要在服务器上设置mysql的相应参数为合适的值就能解决问题。(该参数为max_allowed_packet)

### 2、问题解决方法
#### 2.1、首先登录到mysql数据库上,查看目前max_allowed_packet配置大小
```$xslt
show VARIABLES like '%max_allowed_packet%';
```
默认得到结果如下（说明目前的配置是：1M）：

Variable_name | Value 
-|- 
max_allowed_packet | 1024 
    
#### 2.2、修改max_allowed_packet配置大小（提供以下2种方法）
##### 2.2.1、在mysql配置文件中修改

可以编辑my.cnf来修改（windows下my.ini）,在[mysqld]段或者mysql的server配置段进行修改
```$xslt
max_allowed_packet = 20M
```
如果找不到my.cnf可以通如下命令寻找my.cnf文件位置，linux下该文件在/etc/下
```$xslt
mysql --help | grep my.cnf
```

##### 2.2.2、在mysql命令行中修改

在mysql命令行中运行
```$xslt
set global max_allowed_packet = 2*1024*1024*10
```

#### 2.3、重启mysql服务
- 使用 service 启动：service mysqld restart
  
- 使用 mysqld 脚本启动：/etc/inint.d/mysqld restart

#### 2.4、查看改后配置

在mysql命令行中运行
```$xslt
show VARIABLES like '%max_allowed_packet%';
```
注意：该值设置过小将导致单个记录超过限制后写入数据库失败，且后续记录写入也将失败

### 3、设置了max_allowed_packet参数后,又自动恢复到原来的值
导致这个原因可能有两种情况：
- 服务器内存不够
- 黑客攻击

