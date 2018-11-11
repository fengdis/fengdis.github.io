---
layout:     post
title:      CentOS7 通过yum安装mysql5.7
subtitle:   CentOS7 通过yum安装mysql5.7
date:       2018-11-01
author:     fengdi
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - CentOS
    - MySql
    - Linux
    - 数据库
---

>1.进入到要存放安装包的位置
>
>```
>cd /home/lnmp
>```
>
> 
>
>2.查看系统中是否已安装 MySQL 服务，以下提供两种方式：
>
>```
>rpm -qa | grep mysql
>yum list installed | grep mysql
>```
>
> 
>
>3.如果已安装则删除 MySQL 及其依赖的包：
>
>```
>yum -y remove mysql-libs.x86_64
>```
>
> 
>
>4.下载 mysql57-community-release-el7-8.noarch.rpm 的 YUM 源：
>
>```
>wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
>```
>
> 
>
>5.安装 mysql57-community-release-el7-8.noarch.rpm：
>
>```
>rpm -ivh mysql57-community-release-el7-8.noarch.rpm
>```
>
>安装完后，得到如下两个包：
>
>mysql-community.repo
>mysql-community-source.repo
>
> 
>
>6.安装 MySQL，出现提示的话，一路 Y 到底
>
>```
>yum install mysql-server
>```
>
>安装完毕后，运行mysql，然后在  /var/log/mysqld.log 文件中会自动生成一个随机的密码，我们需要先取得这个随机密码，以用于登录 MySQL 服务端：
>
>```
>service mysqld start
>grep "password" /var/log/mysqld.log
>```
>
>将会返回如下内容，末尾字符串就是密码，把它复制下来：
>
>```
>A temporary password is generated for root@localhost: hilX0U!9i3_6
>```
>
> 
>
>7.登录到 MySQL 服务端并更新用户 root 的密码：
>
>注意：由于 MySQL5.7 采用了密码强度验证插件 validate_password，故此我们需要设置一个有一定强度的密码；
>
>```
>mysql -u root -p
>hilX0U!9i3_6
>```
>
>然后更改密码
>
>```
>SET PASSWORD = PASSWORD('your new password');
>ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
>flush privileges;
>```
>
>设置用户 root 可以在任意 IP 下被访问：
>
>```
>grant all privileges on *.* to root@"%" identified by "new password";
>```
>
>设置用户 root 可以在本地被访问：
>
>```
>grant all privileges on *.* to root@"localhost" identified by "new password";
>```
>
>刷新权限使之生效：
>
>```
>flush privileges;
>```
>
>OK，输入 exit 后用新密码再次登录看看吧！
>
>注意：如果用远程工具还是连接不上，试试用 iptables -F   命令来清除防火墙中链中的规则
>
> 
>
>8.MySQL控制命令：启动、停止、重启、查看状态
>
>```
>service mysqld start
>service mysqld stop
>service mysqld restart
>service mysqld status
>
>systemctl start mysqld
>service mysqld stop
>service mysqld restart
>systemctl status mysqld
>```
>
> 
>
>9.设置 MySQL 的字符集为 UTF-8：
>
>打开 /etc 目录下的 my.cnf 文件（此文件是 MySQL 的主配置文件）：
>
>```
>vim /etc/my.cnf
>```
>
>在 [mysqld] 前添加如下代码：
>
>```
>[client]
>default-character-set=utf8
>```
>
>在 [mysqld] 后添加如下代码：
>
>```
>character_set_server=utf8
>```
>
>**重启mysql后**再登录，看看字符集，6个utf8就算OK
>
>```
>show variables like '%character%';
>```
>
> 
>
>10.查看指定的数据库中指定数据表的字符集，如查看 mysql 数据库中 servers 表的字符集：
>
>```
>show table status from mysql like '%servers%';
>```
>
>查看指定数据库中指定表的全部列的字符集，如查看 mysql 数据库中 servers 表的全部的列的字符集：
>
>```
>show full columns from servers;
>```
>
> 
>
>\11. 忘记密码时，可用如下方法重置：
>
>```
>service mysqld stop
>mysqld_safe --user=root --skip-grant-tables --skip-networking &
>mysql -u root
>```
>
>进入MySQL后
>
>```
>use mysql;
>update user set password=password("new_password") where user="root"; 
>flush privileges;
>```
>
> 
>
>12.一些文件的存放目录
>
>配置文件
>
>```
>vim /etc/my.cnf
>```
>
>存放数据库文件的目录
>
>```
>cd /var/lib/mysql
>```
>
>日志记录文件
>
>```
>vim /var/log/ mysqld.log
>```
>
>服务启动脚本
>
>```
>/usr/lib/systemd/system/mysqld.service
>```
>
>socket文件
>
>```
>/var/run/mysqld/mysqld.pid
>```
>
> 
>
>13.MySQL 采用的 TCP/IP 协议传输数据，默认端口号为 3306，我们可以通过如下命令查看：
>
>```
>netstat -anp
>```
