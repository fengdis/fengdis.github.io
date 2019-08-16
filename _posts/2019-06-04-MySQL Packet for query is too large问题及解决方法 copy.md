---
layout:     post
title:      idea免费永久激活方法
subtitle:   idea免费永久激活方法
date:       2019-08-16
author:     fengdi
header-img: img/post-bg-alibaba.jpg
catalog: true
tags:
    - idea
    - 破解
    - 激活
---

>最近在使用idea的时候，又提示了无法与远程server链接，直接给退出来了，然后网上找了一些注册码，但是发现大部分都不能使用，最后看到了这个方法，主要通过添加补丁来破解，其实也很简单，一共也就三步。



### 1、非永久激活
IntelliJ IDEA 注册码[http://idea.lanyus.com/]，进入网站后点击获取注册码填入idea中即可，此激活时间为一年。
激活前清除hosts中屏蔽域名, 激活后请将“0.0.0.0 account.jetbrains.com”及“0.0.0.0 www.jetbrains.com”添加到hosts文件中

### 2、永久激活
#### 2.1、下载补丁jar包（根据自己的IDEA版本下载对应的破解补丁）,将jar包放入idea安装目录的bin目录下
链接:https://pan.baidu.com/s/1sQ0oF6iT5IVZOdk8CREreg  密码:fz39

#### 2.2、修改idea的bin目录下两个后缀.vmoptions的文件，并在文件最下面添加补丁的路径信息(windows两个文件、macos和linux一个文件)
```$xslt
-javaagent:/Applications/IntelliJ IDEA.app/Contents/bin/JetbrainsIdesCrack-4.2-release-sha1-jfiasf2134123n12342313.jar
```

#### 2.3、复制下面信息，并修改用户名和邮箱信息
```$xslt
ThisCrackLicenseId-{
    "licenseId":"ThisCrackLicenseId",
    "licenseeName":"用户名",
    "assigneeName":"",
    "assigneeEmail":"邮箱信息",
    "licenseRestriction":"For This Crack, Only Test! Please support genuine!!!",
    "checkConcurrentUse":false,
    "products":[
        {"code":"II","paidUpTo":"2099-12-31"},
        {"code":"DM","paidUpTo":"2099-12-31"},
        {"code":"AC","paidUpTo":"2099-12-31"},
        {"code":"RS0","paidUpTo":"2099-12-31"},
        {"code":"WS","paidUpTo":"2099-12-31"},
        {"code":"DPN","paidUpTo":"2099-12-31"},
        {"code":"RC","paidUpTo":"2099-12-31"},
        {"code":"PS","paidUpTo":"2099-12-31"},
        {"code":"DC","paidUpTo":"2099-12-31"},
        {"code":"RM","paidUpTo":"2099-12-31"},
        {"code":"CL","paidUpTo":"2099-12-31"},
        {"code":"PC","paidUpTo":"2099-12-31"}
    ],
    "hash":"2911276/0",
    "gracePeriodDays":7,
    "autoProlongated":false
}
```


