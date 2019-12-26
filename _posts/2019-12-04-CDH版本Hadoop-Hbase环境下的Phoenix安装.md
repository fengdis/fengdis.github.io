---
layout:     post
title:      CDH版本Hadoop-Hbase环境下的Phoenix安装
subtitle:   CDH版本Hadoop-Hbase环境下的Phoenix安装
date:       2019-12-04
author:     fengdi
header-img: img/post-bg-code7.jpg
catalog: true
tags:
    - Hadoop
    - Hbase
    - Phoenix
    - cdh
---

>hadoop环境下列式数据库hbase的关系型数据层

### 1、Phoenix介绍
Phoenix（“凤凰”），它相当于一个Java中间件，提供jdbc连接，操作base数据表。Apache Phoenix是构建在HBase之上的关系型数据库层，作为内嵌的客户端JDBC驱动用以对HBase中的数据进行低延迟访问。

### 2、Phoenix下载
- 如果cdh版本较高的在官网上可以找到对应版本的直接在[官网](http://apache.fayea.com/phoenix/)下载，并且可以跳过后续编译步骤

注意：安装的为cdh版本的hbase必须下载cdh版本的phoenix，phoenix官方版本pom文件里的hbase依赖并不是使用cdh版本的

- 如果cdh版本较低，下载https://github.com/chiastic-security/phoenix-for-cloudera/tree/4.8-HBase-1.2-cdh5.8版本，这里以cdh5.8版本为例，自己编译低版本cdh版本的phoenix

### 3、Phoenix编译
phoenix根目录下执行命令
```$xslt
mvn clean package -DskipTests
```

附：此步骤可能出现的问题

- maven使用阿里源可能下载不到cdh相关的包，在maven的配置中做如下更改（或者直接注释掉阿里源，在phoenix源码pom中有相关源指定）：
```$xslt
    <mirror>  
      <id>nexus-aliyun</id>  
      <mirrorOf>*,!cloudera</mirrorOf>
      <name>Nexus aliyun</name>  
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
    </mirror>
```
增加了!cloudera，意思为可在cloudera官方下载包

- 编译phoenix-hive模块报错，可以在父pom中暂时把phoenix-hive注释掉不编译

### 3、Phoenix配置
编译打包后的phoenix-assembly/target/phoenix-4.8.0-cdh5.8.0.tar.gz解压

#### 3.1、将phoenix-4.8.0-cdh5.8.0-server.jar拷贝到每一个RegionServer下
将解压后目录下的phoenix-4.8.0-cdh5.8.0-server.jar拷贝到每个hbase的lib目录下，因为是伪分布式的，只有一个regionServer所以将指定的phoenix-4.8.0-cdh5.8.0-server.jar复制到hbase的lib文件夹下，只拷贝到一个节点下就可以了，如果你是集群，当然每一个节点都要拷贝一个过去

#### 3.2、增加hbase-site.xml配置
将hbase/conf里的hbase-site.xml复制到phoenix-4.8.0-cdh5.8.0/bin下，覆盖已有的hbase-site.xml

### 3、Phoenix启动
等待hdfs、zookeeper、hbase启动后，进入phoenix/bin下，执行
```$xslt
./sqlline.py localhost:2181/hbase
或
phoenix-sqlline localhost:2181:/hbase
```

    Setting property: [incremental, false]
    Setting property: [isolation, TRANSACTION_READ_COMMITTED]
    issuing: !connect jdbc:phoenix:feng:2181/hbase none none org.apache.phoenix.jdbc.PhoenixDriver
    Connecting to jdbc:phoenix:feng:2181/hbase
    SLF4J: Class path contains multiple SLF4J bindings.
    SLF4J: Found binding in [jar:file:/feng/cloudera/lib/phoenix-4.8.0-cdh5.8.0/phoenix-4.8.0-cdh5.8.0-client.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/feng/cloudera/lib/hadoop-2.6.0-cdh5.7.1/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
    SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
    19/12/04 15:59:25 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    Connected to: Phoenix (version 4.8)
    Driver: PhoenixEmbeddedDriver (version 4.8)
    Autocommit status: true
    Transaction isolation: TRANSACTION_READ_COMMITTED
    Building list of tables and columns for tab-completion (set fastconnect to true to skip)...
    88/88 (100%) Done
    Done
    sqlline version 1.1.9
    
出现如上日志则说明启动并登陆phoenix成功

### 4、测试phoenix
- 列出所有表
```$xslt
!tables
```
- 退出
```$xslt
!quit或!exit
```
- 帮助
```$xslt
help
```
- 列出metadata信息
```$xslt
!dbinfo
```
- 创建表
```$xslt
create table if not exists feng.student(id integer primary key,name varchar(20));
```
- 查询表
```$xslt
select * from feng.student;
```
注意：

1、如果不加双引号，会自动将小写转为大写

2、phoenix表名区分大小写

- 删除表
```$xslt
drop table ljc.student;
```
- 查询表结构
```$xslt
!describe "METRIC_AGGREGATE"
```
注意：phoenix/hbase对表名、字段名都是大小写敏感，如果直接写小写字母，不加双引号，则默认会被转换成大写字母
- 插入更新
```$xslti
upsert into ljc.student(id,name) values(1,'zhangsan');
```
注意：phoenix中不存在update的语法关键字，而是upsert，功能上替代了nsert+update 