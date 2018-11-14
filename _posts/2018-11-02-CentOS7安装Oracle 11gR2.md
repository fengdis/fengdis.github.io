---
layout:     post
title:      CentOS7安装Oracle 11gR2
subtitle:   CentOS7安装Oracle 11gR2
date:       2018-11-02
author:     fengdi
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - CentOS
    - Oracle
    - 数据库
---

>Oracle 在Linux和window上的安装不太一样，公司又是Linux系统上的Oracle，实在没辙，研究下Linux下Oracle的使用，oracle默认不支持CentOS系统安装，所以安装的时候，需要修改部分属性，先参考同行博客和自己安装实践，总结下安装流程。编辑之处，如有缺失，欢迎拍砖....

>1、下载Oracle安装包：[linux.x64_11gR2_database_1of2.zip](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/112010-linx8664soft-100572.html) 和 [linux.x64_11gR2_database_2of2.zip](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/112010-linx8664soft-100572.html) ，可以下载到本地，通过ftp服务上传到Linux系统（[参考CentOS7 FTP服务器搭建](http://www.cnblogs.com/xibei666/p/5934659.html)），也可以使用Linux系统的wget命令，下载文件包；
>
### 2、创建运行oracle数据库的系统用户和用户组：
>用Root账号登录，运行下面指令，创建所需要用户和用户组，[分组原因参考网址](http://www.oracle.com/technetwork/cn/articles/hunter-rac11gr2-iscsi-2-092412-zhs.html#13)
>
>```
>groupadd oinstall　　　　　　　　　　　　　#创建用户组oinstall
>groupadd dba　　      　　　　　　　　　　 #创建用户组dba
>useradd -g oinstall -g dba -m oracle　　#创建oracle用户，并加入到oinstall和dba用户组
>groups oracle  　　　　　　　　　　　　　　#查询用户组是否授权成功
>passwd oracle　　     　　　　　　　　　　 #设置用户oracle的登陆密码，不设置密码，在CentOS的图形登陆界面没法登陆
>id oracle                   　　　　　　 #查看新建的oracle用户
>```
>
### 3、创建oracle数据库安装目录（运行下面指令，创建账号和分配权限）
>
>```
> mkdir -p /data/oracle　　#oracle数据库安装目录
> mkdir -p /data/oraInventory　　#oracle数据库配置文件目录
> mkdir -p /data/database　　#oracle数据库软件包解压目录
> cd /data
> ls　　#创建完毕检查一下
> chown -R oracle:oinstall /data/oracle　　#设置目录所有者为oinstall用户组的oracle用户
> chown -R oracle:oinstall /data/oraInventory
> chown -R oracle:oinstall /data/database
>```

### 4、修改OS系统标识
> oracle默认不支持CentOS系统安装， 修改文件 /etc/[RedHat](http://www.linuxidc.com/topicnews.aspx?tid=10)-release 内容为RedHat-7
>
>```
>vi /etc/redhat-release#修改成红色部分文字
>redhat-7
>```

### 5.安装oracle数据库所需要的软件包
> 以下是按照需要依赖的安装包，通过 yum install {包名} 来验证是否安装，例如yum install binutils
>
>```
>binutils-2.23.52.0.1-12.el7.x86_64 
>compat-libcap1-1.10-3.el7.x86_64 
>gcc-4.8.2-3.el7.x86_64 
>gcc-c++-4.8.2-3.el7.x86_64 
>glibc-2.17-36.el7.i686 
>glibc-2.17-36.el7.x86_64 
>glibc-devel-2.17-36.el7.i686 
>glibc-devel-2.17-36.el7.x86_64 
>ksh
>libaio-0.3.109-9.el7.i686 
>libaio-0.3.109-9.el7.x86_64 
>libaio-devel-0.3.109-9.el7.i686 
>libaio-devel-0.3.109-9.el7.x86_64 
>libgcc-4.8.2-3.el7.i686 
>libgcc-4.8.2-3.el7.x86_64 
>libstdc++-4.8.2-3.el7.i686 
>libstdc++-4.8.2-3.el7.x86_64 
>libstdc++-devel-4.8.2-3.el7.i686 
>libstdc++-devel-4.8.2-3.el7.x86_64 
>libXi-1.7.2-1.el7.i686 
>libXi-1.7.2-1.el7.x86_64 
>libXtst-1.2.2-1.el7.i686 
>libXtst-1.2.2-1.el7.x86_64 
>make-3.82-19.el7.x86_64 
>sysstat-10.1.5-1.el7.x86_64
>```
> 使用下面指令，检查依赖软件包
>```
>yum install binutils-2.* compat-libstdc++-33* elfutils-libelf-0.* elfutils-libelf-devel-* gcc-4.* gcc-c++-4.* glibc-2.* glibc-common-2.* glibc-devel-2.* glibc-headers-2.* ksh-2* libaio-0.* libaio-devel-0.* libgcc-4.* libstdc++-4.* libstdc++-devel-4.* make-3.* sysstat-7.* unixODBC-2.* unixODBC-devel-2.* pdksh*
>```
>
### 6、关闭防火墙和selinux，[具体操作可参考博客](http://www.cnblogs.com/xibei666/p/5934659.html)

### 7、修改内核参数
>```
>vi /etc/sysctl.conf #红色部分是要添加sysctl.conf内容
>
>net.ipv4.icmp_echo_ignore_broadcasts = 1
>net.ipv4.conf.all.rp_filter = 1
>fs.file-max = 6815744 #设置最大打开文件数
>fs.aio-max-nr = 1048576
>kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
>kernel.shmmax = 2147483648 #最大共享内存的段大小
>kernel.shmmni = 4096 #整个系统共享内存端的最大数
>kernel.sem = 250 32000 100 128
>net.ipv4.ip_local_port_range = 9000 65500 #可使用的IPv4端口范围
>net.core.rmem_default = 262144
>net.core.rmem_max= 4194304
>net.core.wmem_default= 262144
>net.core.wmem_max= 1048576
>```

### 8、对oracle用户设置限制，提高软件运行性能（红色为添加部分）
>```
>vi /etc/security/limits.conf  #红色部分要添加到Limits.conf内容
>
>oracle soft nproc 2047
>oracle hard nproc 16384
>oracle soft nofile 1024
>oracle hard nofile 65536
>```

### 9、配置用户的环境变量（红色部分为添加代码）
>```
>vi /home/oracle/.bash_profile  #红色部分是要追加bash_profile内容部分
>
>export ORACLE_BASE=/data/oracle #oracle数据库安装目录
>export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1 #oracle数据库路径
>export ORACLE_SID=orcl #oracle启动数据库实例名
>export ORACLE_TERM=xterm #xterm窗口模式安装
>export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH #添加系统环境变量
>export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib #添加系统环境变量
>export LANG=C #防止安装过程出现乱码
>export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK  #设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致
>```
>
> 配置完成，:wq!保存退出，运行source /home/oracle/.bash_profile时上述配置生效

### 10、获取安装包文件后解压安装包
> 获取安装包文件的方式，[可通过ftp服务器](http://www.cnblogs.com/xibei666/p/5934659.html)，也可通过wget下载到指定目录，解压方式如下
>
>```
>unzip linux.x64_11gR2_database_1of2.zip -d /data/database/  #解压文件1
>unzip linux.x64_11gR2_database_2of2.zip -d /data/database/  #解压文件2
>chown -R oracle:oinstall /data/database/database/　　　　　　 #分配安装文件授权Oracle
>```
>
>Oracle安装
>
>1、oracle用户登录系统，使用命令行跳转到data/database/database目录下，输入./runInstaller 调出安装页面；
>
>![img](https://images2015.cnblogs.com/blog/433460/201610/433460-20161006232532176-49357099.png)
>
>2、调出安装页面，点击下一步进行安装，我选择仅数据库服务安装
>
>![img](https://images2015.cnblogs.com/blog/433460/201610/433460-20161006232627395-1837007543.png)
>
>像window安装Oracle安装一样，此处不再重复介绍。
>
> 安装完成之后，通过netca打开监听配置页面，通过执行dbca命令，启动oracle实例安装界面,一个Oracle服务可以对应多个实例，一个Oracle数据库对应多个表空间和用户名，每个用户名又可管理表空间。
>
> 安装完成实例之后，使用sqlPlus命令链接数据库的时候，提示 could not open parameter file "/data/Oracle/product/11.2/db_1/dbs/initorcl.ora"，这个时候需要将刚刚安装的Oracle实例配置文件（$ORACLE_BASE/admin /数据库名称/pfile目录下的init.ora.012009233838形式的文件）拷贝到/data/Oracle/product/11.2/db_1/dbs目录下
>
>
>[oracle[@localhost](https://my.oschina.net/u/570656) pfile]$ pwd
>/data/oracle/admin/MLUCDB/pfile
>[oracle[@localhost](https://my.oschina.net/u/570656) pfile]$ cp init.ora.962016224738 /data/Oracle/product/11.2/db_1/dbs/initorcl.ora
>
>\#使用sqlplus命令登录Oracle，重启服务器
>
>sqlplus  /nolog
>
>conn / as sysdba;
>
>\#再输入startup，回车.这步是启动oracle服务。
>
>startup
>
>
>
> 重启服务器之后，打开Oracle，提示 ORA-01034: ORACLE not available ORA-27101
>
> 原因在于未启动服务，操作的方式是：
>
>　　1、启动oracle监听：cmd命令行窗口下，输入lsnrctl start，回车即启动监听；
>
>　　2、采用sqlplus /nolog 登录Oracle服务，连接服务conn /as sysdba，然后startup启动服务
>
>扩展RedHat下Oracle的安装
>
>　　1、RedHat系统版本尽量使用Desk版本，便于Oracle的界面安装，Oracle安装文件传输到RedHat服务器，可以通过SecureCrt远程客户端完成数据的传输。
>
>　　2、记得配置用户换机下的安装编码，否则oracle安装会出现乱码：
>
>```
>vi /home/oracle/.bash_profile  #追加配置文件
>export LANG=C #防止安装过程出现乱码
>export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK  #设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致
>```
>
>参考博客
>
>[centos安装oracle 11g 完全图解](http://www.cnblogs.com/zhwl/p/3719302.html) http://www.cnblogs.com/zhwl/p/3719302.html
>
>[CentOS7安装Oracle 11gR2图文详解](http://www.linuxidc.com/Linux/2016-04/130559p3.htm) http://www.linuxidc.com/Linux/2016-04/130559p3.htm
>
>[RedHat.Enterprise.Linux_v6.3系统中安装 Oracle_11gR2教程](http://www.cnblogs.com/swq6413/p/Oracle11gR2_Installation_On_RHEL6.html)  http://www.cnblogs.com/swq6413/p/Oracle11gR2_Installation_On_RHEL6.html
>
>RedHat下载地址：
>
>rhel-server-6.4-x86_64-dvd.iso
>http://pan.baidu.com/s/1dFfoCwx
>
>rhel-server-6.5-x86_64-dvd.iso
>http://pan.baidu.com/s/1eRKnOqe
>
>rhel-server-6.6-x86_64-dvd.iso
>http://pan.baidu.com/s/1o8KnJBk
>
>rhel-server-6.7-x86_64-dvd.iso
>http://pan.baidu.com/s/1i45UBdV
>
>rhel-server-7.0-x86_64-dvd.iso
>http://pan.baidu.com/s/1gfCu7VP
>
>rhel-server-7.1-x86_64-dvd.iso
>http://pan.baidu.com/s/1pLCeo3L
>
>rhel-server-7.2-x86_64-dvd.iso
>http://pan.baidu.com/s/1hsFat4w
>
>http://rhnproxy1.uvm.edu/pub/redhat/rhel6-x86_64/isos/
