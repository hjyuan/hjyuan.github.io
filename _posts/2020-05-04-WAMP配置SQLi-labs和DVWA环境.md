---
layout:     post
title:      WAMP配置SQLi-labs和DVWA环境
subtitle:   web安全环境搭建
date:       2020-05-04
author:     Jie_Y
header-img: img/post-bg-dvwa.jpg
catalog: true
tags:
    - web安全
    - 环境搭建
    - SQL注入
---

# WAMP配置SQLi-labs和DVWA环境

目前正在上Web安全的课程，也跟着i春秋的web安全进阶班进行学习，为了实战就想着搭建相关的web漏洞环境。
因此本文对自己配置环境中遇到的问题和经验进行总结。

## WAMP安装

WAMP是Windows系统下的一款集成环境，其包括了Apache、MySQL/MariaDB、PHP等，常用来搭建动态网站或者服务器等，也用来进行web安全实验平台的搭建，由于安装简单，安装完成后即包含了常用的数据库等环境，因此很受欢迎。

WAMP官网([WAMP](https://www.wampserver.com/))

WAMP安装比较简单，下载完软件后双击打开，语言只有English选项，安装完成后可以更改为中文。  
![setup](/img/post-wamp-setup.gif)  

接受协议进行下一步

![accept](/img/post-wamp-accept.gif)

然后选择安装文件夹，默认是C盘根目录创建WAMP64文件夹，自己可以选择其他地方

![folder](/img/post-wamp-folder.gif)

完成安装后会让选择浏览器，默认是IE，自己可以切换为Chrome等。

![browser](/img/post-wamp-browser.gif)

最后点击完成即安装完成。

## SQLi-labs配置

[SQLi-labs](https://github.com/Audi-1/sqli-labs)是GitHub大神开发的一个专门进行SQL注入漏洞的实验平台

SQLi-labs环境配置相对来说也比较简单，从GitHub上下载源代码，将其解压到WAMP目录下的www目录

![sqli-labs](/img/post-sqli-labs.jpg)

在浏览器输入127.0.0.1，能够看到项目中已经存在SQLi-labs，但是此时还不能使用

![projects](/img/post-sqli-local.jpg)

由于新版本WAMP默认采用的是PHP7.x版本，导致MySQL中有些函数不能使用，因此在配置环境时，需切换PHP到5.x版本。

![version](/img/post-sqli-phpv.jpg)

然后打开PHPMyAdmin，默认账户root，密码为空，修改为自己设置的密码。

![MyAdmin](/img/post-sqli-php.jpg)

打开sqli-labs-master文件夹中的sql-connections文件夹，找到db-creds.inc文件，此文件是连接数据库的账户和密码，找到user和pass、host，将其修改为上一步数据库设置的账户名和密码。

![connection](/img/post-sqli-connection.jpg)

然后打开127.0.0.1/sqli-labs-matster，点击setup/reset Database for labs，出现下面页面即搭建成功。

![success](/img/post-sqli-success.png)

> 注意，在配置时一定要注意将用户名和密码匹配对，并且PHP版本改为5.x版本，否则在进行setup/reset 步骤时会出现如下错误。

```
Uncaught Error: Call to undefined function mysql_connect()
```

经查php手册可知 mysql_connect() 在php5以后的版本中不在使用，使用mysqli_conncet()代替，准确的来说是mysql类被mysqli类代替，在php5+版本中可以同时使用mysql类和mysqli类。  
1、在phpstudy环境下我们可以对php版本进行降级，选择php5+版本即可。  
具体操作操作：打开phpstudy -> 网站 -> 管理 -> php版本 即可。(phpstudy的旧版本可以直接选择更换版本即可)。
如果既想使用php7又不想更改代码可以在管理->php拓展中在php_mysql前勾选即可。

## DVWA配置

[DVWA](http://www.dvwa.co.uk/)（Damn Vulnerable Web App）是一个基于PHP/MySql搭建的Web应用程序，旨在为安全专业人员测试自己的专业技能和工具提供合法的环境，帮助Web开发者更好的理解Web应用安全防范的过程。

在DVWA官网点击github，找到最新的稳定版，将其下载后解压到www目录下，命名为DVWA。

![local](/img/post-dvwa-local.jpg)

进入文件夹找到config文件夹中的config.inc.php.dist文件，将其命名为config.inc.php，打开文件将其中的db_user和db_password**修改**为配置WAMP数据库时的账户和密码

![user](/img/post-dvwa-user.jpg)

访问127.0.0.1/DVWA，点击下方的Create/Reset Database按钮，就会创建成功

![success](/img/post-dvwa-success.png)

然后点击login，会跳转到登陆界面，默认账户是admin，密码是password，成功登录。

> 注意，config.inc.php中的账户和密码一定要和MySQL数据库的账户和密码匹配，否则会出现下面错误

![problem](/img/post-dvwa-problem.png)

