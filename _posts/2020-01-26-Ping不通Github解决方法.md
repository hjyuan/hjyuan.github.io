---
layout:     post
title:      Ping不通Github解决方法
subtitle:   Ping Github
date:       2020-01-26
author:     Jie_Y
header-img: img/post-bg-Github.jpg
catalog: true
tags:
    - Ping
    - Github
    - 环境配置
---

## 出错

今天在家无事，学了一点知识想要发布到GitHub上面，但是写好以后发布时显示错误，无法连接到GitHub。

使用git push的时候提示"fatal: unable to access 'https://github.com/xxxxx/xxxx.git/': Failed to connect to github.com port 443: Timed out"

然后又尝试ping GitHub的网址，结果显示连接超时。

上网查了查，找到了两种解决办法。

### 1、更改代理

由于之前使用酸酸乳和clash进行了科学上网，设置了代理，导致断开连接后无法进行GitHub访问。

可以用git来更改代理。

`git config --global http.proxy 127.0.0.1:1080`  
`git config --global https.proxy http://127.0.0.1:1080`

一个是http协议，一个是https协议，端口设置为代理的端口。

​    但是这种方法只能在使用代理的情况下，一旦代理断开就无法访问

### 2、更改hosts

找到C:\Windows\System32\drivers\etc(Linux系统在/etc/hosts)下的hosts文件，以管理员身份打开，在最后添加几行，就可以正常访问。

`192.30.253.113    github.com `  
`192.30.252.131 github.com `  
`185.31.16.185 github.global.ssl.fastly.net `  
`74.125.237.1 dl-ssl.google.com `  
`173.194.127.200 groups.google.com `  
`192.30.252.131 github.com `  
`185.31.16.185 github.global.ssl.fastly.net `  
`74.125.128.95 ajax.googleapis.com`  

然后就能够push和ping了。