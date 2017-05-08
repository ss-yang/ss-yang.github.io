---
title: centos7 创建应用程序快捷启动器的方法
date: 2017-04-30 10:00:27
tags: linux
categories: linux
comments: true
---

## 原因
在linux下装好了android studio, 却发现只能在安装目录下通过terminal执行studio.sh来运行，这样明显很不方便。
但作为一个linux新司机，并不知道怎么办。
<!--more-->
桌面右建->新建启动器，centos里根本没有这一项。
后来因为某些原因需要重装中国版的firefox，官方安装包下载下来后，也是直接解压使用的(需要从terminal启动）。
同一个问题遇到两次，不能忍。

## 解决方法
查资料后得知，**centos 的程序图表都在/usr/share/applications/文件夹下，里面都是.desktop文件**。
于是问题迎刃而解。

* 新建一个android-studio.desktop文件，内容如下：

    
    [Desktop Entry]
    Encoding=UTF-8
    Name=Android Studio 2.3
    Comment=Android Studio
    Comment[zh_CN]=安卓开发
    Exec=/usr/local/bin/android-studio #运行路径
    Icon=/opt/android-studio/bin/studio.png #图标
    Categories=Application;Development;Android; #分类：这个将在应用的编程分类中
    Actions=NewWindow;NewPrivateWindow;
    Type=Application
    Terminal=0 #是否命令行形式


至此，应用程序菜单里面就有了刚刚添加的启动器了，也可以将desktop文件复制到桌面。

