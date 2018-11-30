---
layout:     post
title:      linux系统下redis的安装和部署
subtitle:   linux系统下redis的安装和部署
date:       2018-11-28
author:     fengdi
header-img: img/post-bg-code8.jpeg
catalog: true
tags:
    - linux
    - redis
---

### 1、安装

#### 1.1、下载
先到redis官网(https://redis.io/download)下载redis安装包
- linux采用命令下载：

    wget http://download.redis.io/releases/redis-4.0.1.tar.gz

- windows下载上传至linux上

#### 1.2、解压
解压并进入redis目录
    tar -zxvf redis-4.0.9
    cd redis-4.0.9/

#### 1.3、编译
通过make命令编译，如果编译的时候报gcc命令找不到的话，可以通过下面的命令安装gcc命令，gcc是c的编译命令
    
    yum install gcc-c++

make是自动编译，会根据Makefile中描述的内容来进行编译。
编译后可以看到在src目录下生成了几个新的文件。
    
    make

    [root@localhost redis]# ll -tr src
    总用量 44440
    -rw-rw-r--. 1 root root    3779 7月  24 22:58 zmalloc.h
    .
    .
    .
    -rw-r--r--. 1 root root   56148 9月   3 10:11 rax.o
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:11 redis-server
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:11 redis-sentinel
    -rw-r--r--. 1 root root  143800 9月   3 10:11 redis-cli.o
    -rwxr-xr-x. 1 root root 5092431 9月   3 10:11 redis-cli
    -rw-r--r--. 1 root root   44892 9月   3 10:11 redis-benchmark.o
    -rwxr-xr-x. 1 root root 4985275 9月   3 10:11 redis-benchmark
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:11 redis-check-rdb
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:11 redis-check-aof
    [root@localhost redis]#

为了使用方便，我们需要将这个几个文件加到/usr/local/bin目录下去。这个目录在Path下面的话，就可以直接执行这几个命令了。执行make install命令：
    
    make install

    [root@localhost redis]# make install
    cd src && make install
    make[1]: Entering directory `/usr/local/redis/src'
        CC Makefile.dep
    make[1]: Leaving directory `/usr/local/redis/src'
    make[1]: Entering directory `/usr/local/redis/src'

    Hint: It's a good idea to run 'make test' ;)

        INSTALL install
        INSTALL install
        INSTALL install
        INSTALL install
        INSTALL install
    make[1]: Leaving directory `/usr/local/redis/src'
    [root@localhost redis]# cd ..
    [root@localhost local]# ll
    总用量 44
    drwxr-xr-x. 2 root root 4096 9月   3 10:16 bin
    drwxr-xr-x. 2 root root 4096 9月   3 10:04 data
    drwxr-xr-x. 2 root root 4096 9月  23 2011 etc
    drwxr-xr-x. 2 root root 4096 9月  23 2011 games
    drwxr-xr-x. 2 root root 4096 9月  23 2011 include
    drwxr-xr-x. 2 root root 4096 9月  23 2011 lib
    drwxr-xr-x. 2 root root 4096 9月  23 2011 libexec
    drwxrwxr-x. 6 root root 4096 7月  24 22:58 redis
    drwxr-xr-x. 2 root root 4096 9月  23 2011 sbin
    drwxr-xr-x. 5 root root 4096 4月   1 04:48 share
    drwxr-xr-x. 2 root root 4096 9月  23 2011 src
    [root@localhost local]# ll bin
    总用量 30908
    -rwxr-xr-x. 1 root root 4985275 9月   3 10:16 redis-benchmark
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:16 redis-check-aof
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:16 redis-check-rdb
    -rwxr-xr-x. 1 root root 5092431 9月   3 10:16 redis-cli
    lrwxrwxrwx. 1 root root      12 9月   3 10:16 redis-sentinel -> redis-server
    -rwxr-xr-x. 1 root root 7185836 9月   3 10:16 redis-server
    [root@localhost local]#

可以看到，这几个文件就已经被加载到bin目录下了

到此就安装完成。但是，由于安装redis的时候，我们没有选择安装路径，故是默认位置安装。在此，我们可以将可执行文件和配置文件移动到习惯的目录。 

    cd /usr/local 
    mkdir -p /usr/local/redis/bin 
    mkdir -p /usr/local/redis/etc 
    cd /usr/local/redis-4.0.2 
    mv ./redis.conf /usr/local/redis/etc 
    cd src 
    mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server redis-sentinel /usr/local/redis/bin

如果想要安装到固定目录下，执行如下命令：

    make install PREFIX=/usr/local/redis

之后需要将配置文件移动到redis安装目录：

    mv redis.conf /usr/local/redis/etc/

### 2、启动、连接、关闭
**比较重要的可执行文件（/usr/local/redis/bin）：**
- redis-server：Redis服务器程序 
- redis-cli：Redis客户端程序，它是一个命令行操作工具。也可以使用telnet根据其纯文本协议操作。 
- redis-benchmark：Redis性能测试工具，测试Redis在你的系统及配置下的读写性能
- redis-check-aof：检查aof日志的工具
- redis-check-dump：检查rdb日志的工具

#### 2.1、启动

    /usr/local/redis/bin/redis-server 
    或 
    cd /usr/local/redis/bin 
    ./redis-server /usr/local/redis/etc/redis.conf

#### 2.2、客户端连接
    /usr/local/redis/bin/redis-cli 

#### 2.3、关闭
    /usr/local/redis/bin/redis-cli shutdown
    或
    pkill redis-server

#### 2.4、开机自启
    vim /etc/rc.local
    加入
    /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis-conf

    ps aux | grep redis 查看redis是否启动成功
    netstat -tlun 查看主机的6379端口是否在使用（监听）

### 3、配置

**下面列举了Redis中的一些常用配置项：**

- daemonize：如需要在后台运行，把该项的值改为yes

- pdifile：把pid文件放在/var/run/redis.pid，可以配置到其他地址

- bind：指定redis只接收来自该IP的请求，如果不设置，那么将处理所有请求，在生产环节中最好设置该项

- port：监听端口，默认为6379

- timeout：设置客户端连接时的超时时间，单位为秒

- loglevel：等级分为4级，debug，revbose，notice和warning。生产环境下一般开启notice

- logfile：配置log文件地址，默认使用标准输出，即打印在命令行终端的端口上

- database：设置数据库的个数，默认使用的数据库是0

- save：设置redis进行数据库镜像的频率

- rdbcompression：在进行镜像备份时，是否进行压缩

- dbfilename：镜像备份文件的文件名

- dir：数据库镜像备份的文件放置的路径

- slaveof：设置该数据库为其他数据库的从数据库

- masterauth：当主数据库连接需要密码验证时，在这里设定

- requirepass：设置客户端连接后进行任何其他指定前需要使用的密码

- maxclients：限制同时连接的客户端数量

- maxmemory：设置redis能够使用的最大内存

- appendonly：开启appendonly模式后，redis会把每一次所接收到的写操作都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态

- appendfsync：设置appendonly.aof文件进行同步的频率

- vm_enabled：是否开启虚拟内存支持

- vm_swap_file：设置虚拟内存的交换文件的路径

- vm_max_momery：设置开启虚拟内存后，redis将使用的最大物理内存的大小，默认为0

- vm_page_size：设置虚拟内存页的大小

- vm_pages：设置交换文件的总的page数量

- vm_max_thrrads：设置vm IO同时使用的线程数量


### 4、卸载
#### 4.1、停止redis服务
首先，通过下面的命令查看redis服务是否在运行

    [root@localhost ~]# ps aux|grep redis
    root      2553  0.2  0.1  41964  1916 ?        Ssl  09:38   0:00 redis-server 127.0.0.1:6379
    root      2565  0.0  0.0   6048   780 pts/0    S+   09:39   0:00 grep redis
    [root@localhost ~]#

可以看到，在6379端口，有redis-server的监听。
通过下面的命令停止redis服务

    [root@localhost ~]# redis-cli shutdown
    [root@localhost ~]# ps aux|grep redis
    root      2575  0.0  0.0   6048   780 pts/0    S+   09:41   0:00 grep redis
    [root@localhost ~]#

可以看到，已经停止了redis服务了。

需要注意的是，由于我的redis命令都安装到/usr/local/bin目录下面了，并且添加到环境变量PATH里面了，所以可以直接运行。

#### 4.2、删除make的时候生成的几个redis*的文件

    [root@localhost local]# ll /usr/local/bin
    总用量 30908
    -rwxr-xr-x. 1 root root 4985307 9月   2 21:13 redis-benchmark
    -rwxr-xr-x. 1 root root 7185872 9月   2 21:13 redis-check-aof
    -rwxr-xr-x. 1 root root 7185872 9月   2 21:13 redis-check-rdb
    -rwxr-xr-x. 1 root root 5092475 9月   2 21:13 redis-cli
    lrwxrwxrwx. 1 root root      12 9月   2 21:13 redis-sentinel -> redis-server
    -rwxr-xr-x. 1 root root 7185872 9月   2 21:13 redis-server
    [root@localhost local]# rm -f /usr/local/bin/redis*
    [root@localhost local]# ll /usr/local/bin
    总用量 0
    [root@localhost local]#

#### 4.3、删除掉解压后的文件目录和所有文件

    [root@localhost local]# ll
    总用量 40
    drwxr-xr-x. 2 root root 4096 9月   3 09:43 bin
    drwxr-xr-x. 2 root root 4096 9月  23 2011 etc
    drwxr-xr-x. 2 root root 4096 9月  23 2011 games
    drwxr-xr-x. 2 root root 4096 9月  23 2011 include
    drwxr-xr-x. 2 root root 4096 9月  23 2011 lib
    drwxr-xr-x. 2 root root 4096 9月  23 2011 libexec
    drwxrwxr-x. 6 root root 4096 9月   2 21:11 redis
    drwxr-xr-x. 2 root root 4096 9月  23 2011 sbin
    drwxr-xr-x. 5 root root 4096 4月   1 04:48 share
    drwxr-xr-x. 2 root root 4096 9月  23 2011 src
    [root@localhost local]# rm -rf redis
    [root@localhost local]# ll
    总用量 36
    drwxr-xr-x. 2 root root 4096 9月   3 09:43 bin
    drwxr-xr-x. 2 root root 4096 9月  23 2011 etc
    drwxr-xr-x. 2 root root 4096 9月  23 2011 games
    drwxr-xr-x. 2 root root 4096 9月  23 2011 include
    drwxr-xr-x. 2 root root 4096 9月  23 2011 lib
    drwxr-xr-x. 2 root root 4096 9月  23 2011 libexec
    drwxr-xr-x. 2 root root 4096 9月  23 2011 sbin
    drwxr-xr-x. 5 root root 4096 4月   1 04:48 share
    drwxr-xr-x. 2 root root 4096 9月  23 2011 src
    [root@localhost local]#

到此，redis卸载完成。