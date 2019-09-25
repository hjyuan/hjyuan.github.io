---
layout:     post
title:      VS code配置python环境
subtitle:   VScode python
date:       2019-09-24
author:     Jie_Y
header-img: img/post-bg-vscode.jpg
catalog: true
tags:
    - VScode
    - python
    - 环境配置
---


# 介绍
>之前一直用pycharm进行python代码编写开发，小辣鸡要用到VS code进行python代码编写，VS code主要在于轻便，并且风格也比较好，在这里进行对VS code配置python环境的介绍。

Visual Studio Code (简称 VS Code / VSC) 是一款免费开源的现代化轻量级代码编辑器，支持几乎所有主流的开发语言的语法高亮、智能代码补全、自定义快捷键、括号匹配和颜色区分、代码片段、代码对比 Diff、GIT命令 等特性，支持插件扩展，并针对网页开发和云端应用开发做了优化。软件跨平台支持 Win、Mac 以及 Linux，运行流畅，可谓是微软的良心之作......

# 步骤
1. 安装python

   直接官网(http://www.python.org/)下载python3.7版本进行默认安装即可，另外注意安装时勾选配置python环境。

2. 安装VS code

   直接官网(http://code.visualstudio.com/)下载VS code最新版，默认安装即可。

3. 安装插件

   打开软件后，点击左边最下面图标，搜索python，选择列表第一个插件并点击install安装程序。
   
   ![](/img/post-art-vscode1.jpg)

4. 选择python解释器

   打开命令面板(Ctrl+Shift+P)输入Python:select Interpreter然后选择一个解释器。

   ![](/img/post-art-vscode2.jpg)

5. 打开工作目录

点击左边的文件图标，再点击"open folder"按钮，选择一个文件夹作为工作目录，之后新建的文件都会存放在这个目录下。

![](/img/post-art-vscode3.png)

![](/img/post-art-vscode4.jpg)

6. 添加配置

选择左边第三个图标，再点击"Add Configuration"添加配置文件，并选择Python选项。

![](/img/post-art-vscode5.jpg)

选择python后会生成python的配置文件launch.json，加入python的安装目录：<"python.pythonPath":"C:/Users/***/AppData/Local/Programs/Python/Python37",>。

![](/img/post-art-vscode6.jpg)

7. 添加用户配置

点击File->Prefrences->Settings，生成一个"User Settings"文件，填入Python安装目录：<"python.pythonPath":"C:/Users/***/AppData/Local/Programs/Python/Python37",>，如下图所示

![](/img/post-art-vscode7.jpg)   
![](/img/post-art-vscode8.jpg)

8. 新建hello.py

点击左边文件的图标，鼠标移动到工程的目录名，会出来一个+号的文件，点击，然后输入hello.py会生成.py文件，点击F5，调试窗口会运行结果，没有报错即成功。

![](/img/post-art-vscode9.jpg)


# 参考
1. https://blog.csdn.net/vinkim/article/details/81546333
2. https://www.cnblogs.com/gitwow/p/10348675.html