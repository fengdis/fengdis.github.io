---
layout:     post
title:      本地git上传代码到github等远程服务空间
subtitle:   本地git上传代码到github等远程服务空间
date:       2019-03-31
author:     fengdi
header-img: img/post-bg-others7.jpg
catalog: true
tags:
    - git
    - github
---

### 1、本地安装git客户端
下载git客户端，傻瓜式安装即可。下载地址https://git-scm.com/downloads

### 2、配置本地git
##### 2.1、初始化用户名、邮箱
打开你要放置项目的本地路径，右键选择$ Git Bash Here，输入如下命令：
```aidl
$ git config --global --replace-all user.name "用户名" 
$ git config --global --replace-all user.email "邮箱地址"
```

##### 2.2、检测当前电脑是否配置ssh
```aidl
cd ~/.ssh
```
如果没有出现：No such file or directory 这句话说明存在ssh配置，接着清理原有ssh密钥：
```aidl
$ mkdir key_backup
$ cp id_rsa* key_backup
$ rm id_rsa*
```

##### 2.3、生成密匙
```aidl
$ ssh-keygen -t rsa -C "邮箱"
```
出现如下代码片段

    Generating public/private rsa key pair.
    Enter file in which to save the key (/your_home_path/.ssh/id_rsa):
    
可以直接按Enter跳过，无需设置；接下来是让你输入做提交代码之类的操作的时候的密码，根据个人需要，如果要设置密码，直接输入密码，按Enter确认再输入，如果不设置直接按两次Enter跳过即可。

附：
此步骤可以再git的图形化界面中完成。右键单击桌面空白处，选择Git Gui Here，进去之后，选择左上角的help选项，会出现一个Show SSH Key，然后点击“Generate Key”得到密匙。

### 3、将密匙添加到github远程服务中
生成完毕之后打开你的电脑 C:\Users\Administrator\.ssh 文件夹下id_rsa.pub文件，复制里面的内容，或者上一步在Git Gui Here中“Generate Key”后获得的内容
打开github，登陆后，打开设置界面，在SSH Keys栏中点击“Add SSH key”按钮，然后填入上边复制的内容，这时候配置完毕。可以再git命令窗口中执行操作了。

### 4、git命令

    git clone "SSH地址"                                      clone
    git clone -b "分支名" "SSH地址"                          clone某个分支
    git push                                                 上传代码至服务器仓库
    git pull                                                 下载服务器仓库代码
    git add .                                                添加当前目录下的所有文件到本地暂存区
    git commit -m '注释'                                     添加提交信息，提交代码到本地仓库
    
    