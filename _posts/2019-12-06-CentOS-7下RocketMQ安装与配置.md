---
layout:     post
title:      CentOS-7下RocketMQ安装与配置
subtitle:   CentOS-7下RocketMQ安装与配置
date:       2018-12-06
author:     fengdi
header-img: img/post-bg-code5.jpg
catalog: true
tags:
    - RocketMQ
    - 消息队列
    - 中间件
---

### 1、下载安装包
可以选在在windows上提前下载，sftp到linux上，或者使用wget直接在linux上下载

    wget http://mirrors.hust.edu.cn/apache/rocketmq/4.2.0/rocketmq-all-4.3.2-bin-release.zip

### 2、安装
我们下载的是编译后的版本，省略编译步骤，直接创建一个rocketmq的文件夹

    mkdir -p /usr/local/rocketmq
解压：

    unzip rocketmq-all-4.3.2-bin-release.zip -d /usr/local/rocketmq

注意：如果解压unzip报错command not found的，请安装unzip，安装之后再次执行解压步骤
​    
    yum install unzip

### 3、环境测试
首先进入rocketmq文件夹中：

    cd /usr/local/rocketmq/

测试启动nameserver

    nohup sh bin/mqnamesrv &

此步骤后看到nohup（nohup 是永久执行、
& 是指在后台运行）正常执行启动，查看rocketmq根目录hs_err_pidxxxxx.log文件，其中出现以下错误：

![](https://img-blog.csdn.net/20180713114627775?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NkbmlnaHQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

内存不足，测试环境调低一点内存容量：

    vim bin/runbroker.sh

-server -Xms8g -Xmx8g -Xmn4g修改为-server -Xms256m -Xmx256m -Xmn128m

JAVA_OPT=”${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m”

    vim bin/runserver.sh

-server -Xms4g -Xmx4g -Xmn2g修改为-server -Xms256m -Xmx256m -Xmn128m

JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m"

    bin/mqnamesrv

发现测试通过

### 4、设置环境变量
    设置环境变量
其实不设置环境变量也可以，但是我们为了进一步简化后续的broker集群命令，所以还是设置一下吧。
配置rocketmq的环境变量
vim /etc/profile

在结尾添加
#设置rocketmq的环境变量

    export ROCKETMQ_HOME=/usr/local/rocketmq
    export PATH=$JAVA_HOME/bin:$ROCKETMQ_HOME/bin:$PATH
    
    使环境变量生效
    source /etc/profile

### 5、启动服务
#### 5.1、启动NameServer

    nohup sh bin/mqnamesrv &
    tail -f ~/logs/rocketmqlogs/namesrv.log

日志中出现如下信息则启动成功：

    The Name Server boot success......

#### 5.2、启动Broker

    nohup sh bin/mqbroker -n localhost:9876 &
    tail -f ~/logs/rocketmqlogs/broker.log

日志中出现如下信息则启动成功：

    The broker[%s, 172.30.30.233:10911] boot success...

### 6、关闭服务

    sh mqshutdown namesrv
    sh mqshutdown broker

### 7、安装RocketMQ可视化管理控制台rocketmq-console-ng

RocketMQ有一个对其扩展的开源项目incubator-rocketmq-externals，这个项目中有一个子模块叫“rocketmq-console”，这个便是管理控制台项目了。

先将incubator-rocketmq-externals拉到本地，因为我们需要自己对rocketmq-console进行编译打包运行，这里演示在windows下mvn编译

![](https://img-blog.csdn.net/20170524113000007?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过命令行进入到rocketmq-console子目录，通过maven对其进行编译打包，

    mvn package

如下图：

![](https://img-blog.csdn.net/20170524113544298?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

打包成功后命令行如下图所示：

![](https://img-blog.csdn.net/20170524113636823?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

此时在rocketmq-console/target目录下生成了一个叫rocketmq-console-ng-1.0.0.jar的jar包，如下图：

![](https://img-blog.csdn.net/20170524113903381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

接下来运行这个jar包，我们可以直接通过java -jar的方式运行，但为了方便，我是将java -jar命令编写成脚本，方便以后直接运行即可。

- windows下

新建一个rocketmq-console-ng.bat文件（与上面生成的rocketmq-console-ng-1.0.0.jar在同一个目录），内容如下：

    @echo off
    java -jar rocketmq-console-ng-1.0.0.jar --server.port=12581 --rocketmq.config.namesrvAddr=10.89.0.64:9876;10.89.0.65:9876
    @pause

-linux下
新建一个rocketmq-console-ng.sh文件

    java -jar rocketmq-console-ng-1.0.0.jar --server.port=12581 --rocketmq.config.namesrvAddr=10.89.0.64:9876;10.89.0.65:9876

这里注意需要设置两个参数：--server.port为运行的这个web应用的端口，如果不设置的话默认为8080；--rocketmq.config.namesrvAddr为RocketMQ命名服务地址，如果不设置的话默认为“”。

设置好后就可以直接双击运行脚本文件，如下图：

![](https://img-blog.csdn.net/20170524140852900?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


启动成功后，我们就可以通过浏览器访问http://localhost:12581进入控制台界面了，如下图：

![](https://img-blog.csdn.net/20170524141327170?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF5ampi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至此全部完成。





