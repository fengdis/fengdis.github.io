---
layout:     post
title:      CentOS7通过yum安装svn服务
subtitle:   CentOS7通过yum安装svn服务
date:       2018-10-25
author:     fengdi
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - CentOS
    - svn
    - 版本控制
---

>svn服务器搭建

### 1、yum install subversion

### 2、输入rpm -ql subversion查看安装位置

### 3、创建svn版本库目录
>mkdir -p /var/svn/svnrepos

### 4、创建版本库
>svnadmin create /var/svn/svnrepos

### 5、进入conf目录（该svn版本库配置文件）
>authz    文件是权限控制文件
>passwd    是帐号密码文件
>svnserve.conf    SVN服务配置文件

### 6、设置账号密码
>vi passwd
>在[users]块中添加用户和密码，格式：帐号=密码，如admin=admin

### 7、设置权限
>vi authz
>在末尾添加如下代码：
>[/]
>admin=rw
>feng=r
>意思是版本库的根目录admin对其有读写(rw)权限，feng只有读(r)权限

### 8、修改svnserve.conf文件
>vi svnserve.conf
>打开下面的几个注释：

>anon-access = read #匿名用户可读
>auth-access = write #授权用户可写
>password-db = passwd #使用哪个文件作为账号文件
>authz-db = authz #使用哪个文件作为权限文件
>realm = /var/svn/svnrepos # 认证空间名，版本库所在目录

### 9、启动svn版本库
>svnserve -d -r /var/svn/svnrepos

### 10、SVN默认的打开端口是3690
>可以通过下面的命令查看：
>netstat -antp | grep svn
>tcp        0      0 0.0.0.0:3690            0.0.0.0:*               LISTEN      66486/svnserve 

### 11、centos7 打开防火墙端口
>$ sudo firewall-cmd --permanent --add-port=3690/tcp
>$ sudo firewall-cmd --reload
