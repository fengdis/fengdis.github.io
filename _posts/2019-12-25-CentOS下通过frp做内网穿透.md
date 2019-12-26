---
layout:     post
title:      CentOS7下通过frp做内网穿透
subtitle:   CentOS7下通过frp做内网穿透
date:       2019-12-25
author:     fengdi
header-img: img/post-bg-code5.jpg
catalog: true
tags:
    - frp
    - 内网穿透
---

>近期服务器经常被暴力扫描、攻击，故花时间处理服务器端口暴力扫描问题，将一些可能存在的风险处理掉. 以下从内网穿透方面解决问题

### 1、前言
 - 为阻止服务器被暴力扫描、攻击，防火墙可以采用白名单策略, 只开放必要端口. 如80/443以及ssh登录端口。
而数据库、缓存等端口, 最好只允许本地访问, 若需要调试, 建议使用白名单IP、代理等方法进行连接.

 - 对于有调试需求的服务器而言, 数据库、缓存、消息队列等端口需要开放. 而直接开放端口会给服务器带来不必要的安全隐患. 此时我们必须对访问者进行限制, 如: IP白名单、VPN等. 除此之外, 笔者推荐一个内网穿透工具用来辅助建立调试环境 —— FRP
 - FRP是一个可用于内网穿透的高性能的反向代理应用, 有多种穿透方式, 可以指定端口进行代理. 我们可以在服务器启动服务端(frps)和客户端(frpc)两个服务, 本地客户端的frpc通过frps监听的唯一端口与服务端的frpc建立连接, 这样就能将服务器上只能内部访问的端口映射到开发者电脑本地端口.
 - 使用FRP的优势: 指定端口、可压缩流量、可加密、服务端只需要暴露frps的端口.

### 2、下载
      
    $ wget https://github.com/fatedier/frp/releases/download/v0.21.0/frp_0.21.0_linux_amd64.tar.gz
    $ tar -xzvf frp_0.21.0_linux_amd64.tar.gz

解压之后的文件夹中既包含了服务端的文件又包括客户端的文件，所以可以分别在两个机器上删除掉不必要的文件，也可以不删，都没有影响。强迫症还是来删一下，在解压后的文件夹中：

- 在公网服务器删除客户端相关的文件，只保留以下两个文件：
  
    frps  frps.ini
    
- 在内网机器上删除服务端相关的文件，只保留以下两个文件：

    frpc  frpc.ini

    
### 3、配置
就是需要修改配置文件 frps.ini 及 frpc.ini
- 修改公网服务器上的服务端配置文件 frps.ini，如下：

```$xslt
[common]
bind_port = 7000        #frp服务端端口（必须）
```

- 修改内网目标主机的客户端配置文件 frpc.ini，如下：

```$xslt
[common]
server_addr = xxx.xxx.xxx.xxx   #frp服务端地址，必须是公网ip或者域名，这里假设为xxx.xxx.xxx.xxx
server_port = 7000      #frp服务端端口，即填写服务端配置中的 bind_port

[ssh]
type = tcp              #连接类型，填tcp或udp
local_ip = 127.0.0.1    #填127.0.0.1或内网ip都可以
local_port = 22         #需要转发到的端口，ssh端口是22
remote_port = 6000      #frp服务端的远程监听端口，即你访问服务端的remote_port就相当于访问客户端的 local_port，如果填0则会随机分配一个端口
```

### 4、运行
- 在公网服务器上运行服务端程序：

```$xslt
$ nohup ./frps -c frps.ini &

$ tail -f nohup.out
2018/09/17 21:34:01 [I] [service.go:130] frps tcp listen on 0.0.0.0:7000
2018/09/17 21:34:01 [I] [root.go:207] Start frps success
2018/09/17 22:06:02 [I] [service.go:319] client login info: ip [125.71.229.32:60516] version [0.21.0] hostname [] os [linux] arch [amd64]
2018/09/17 22:06:02 [I] [proxy.go:217] [7940291c148c2fca] [ssh] tcp proxy listen port [6000]
2018/09/17 22:06:02 [I] [control.go:335] [7940291c148c2fca] new proxy [ssh] success
```

- 在内网目标主机上运行客户端程序：

```$xslt
$ nohup ./frpc -c frpc.ini &

$ tail -f nohup.out
2018/09/17 22:42:22 [I] [proxy_manager.go:300] proxy removed: []
2018/09/17 22:42:22 [I] [proxy_manager.go:310] proxy added: [ssh1]
2018/09/17 22:42:22 [I] [proxy_manager.go:333] visitor removed: []
2018/09/17 22:42:22 [I] [proxy_manager.go:342] visitor added: []
2018/09/17 22:42:23 [I] [control.go:246] [0624b332c3465118] login to server success, get run id [0624b332c3465118], server udp port [0]
2018/09/17 22:42:23 [I] [control.go:169] [0624b332c3465118] [ssh1] start proxy success
```

### 5、配置多个客户端
内网机器1和内网机器2的配置应该区分如下：

```$xslt
内网机器1：
[ssh]                      <==不同点
type = tcp 
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000         <==不同点

内网机器2：
[ssh1]                     <==不同点
type = tcp 
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001         <==不同点
```

在两个内网机器上分别运行 frpc 客户端程序后，一般就可以通过以下的方法 ssh 登录：

```$xslt
内网机器1：
$ ssh -p 6000 user_name1@server_addr

内网机器2：
$ ssh -p 6001 user_name2@server_addr
```
以上参数中，server_addr是公网服务器的公网ip；user_name1、user_name2 分别是内网机器1、2的用户名，之后分别使用登录密码就可以登录。

### 6、connection timed out 解决
有时候会发现按照以上的配置还是使 frp 的服务端与客户端建立连接，在客户端上会出现以下错误：

```$xslt
2018/09/17 22:02:23 [W] [control.go:113] login to server failed: dial tcp xxx.xxx.xxx.xxx:7000: connect: connection timed out
dial tcp xxx.xxx.xxx.xxx:7000: connect: connection timed out
```
从以下两个方面检查问题：
- 服务器防火墙
服务器放行frp服务端口、代理端口

```$xslt
开放端口
firewall-cmd --zone=public --add-port=7000/tcp --permanent

查看开放端口列表
firewall-cmd --permanent --zone=public --list-ports

防火墙reload
firewall-cmd --reload
```

- 云服务器安全组规则
云服务器放行frp服务端口、代理端口
云服务器厂商安全组主要有两个配置项，端口范围和授权对象

    1、端口范围就是你要开放的端口的范围，需要将 server_port 和 remote_port 都添加进去，可以分成两条规则分别添加；

    2、授权对象是你要授权的对象，一般设为 0.0.0.0/0 表示允许所有IP的访问。
