---
title: "优雅地使用ubuntu18之二"
date: 2019-06-10
draft: false
categories: ["操作系统"]
---


## 文章链接
[优雅地使用ubuntu18.04（一）](https://blog.csdn.net/wjh2622075127/article/details/91182584)
[优雅地使用ubuntu18.04（二）](https://blog.csdn.net/wjh2622075127/article/details/91384365)
## 12、设置默认终端
使用命令`sudo update-alternatives --config x-terminal-emulator`
然后就可以选择了
![](/images/20190610_1.png)

## 13、设置开启自动启动
找到`启动应用程序`

![](/images/20190610_2.png)
就可以直接添加自启动了
![](/images/20190610_3.png)

## 14、自定义右键
想实现文件夹右键，可以执行打开某一指定程序（如我们下载的终端），并设置工作目录为当前目录。
但还没有实现，挖个坑。
## 15、使用albert
这个工具类似于windows中的everything
但感觉不是很好用的样子？虽然有人强烈推荐
[可以直接下载deb文件进行安装了](https://software.opensuse.org/download.html?project=home:manuelschneid3r&package=albert)

## 16、设置终端复制黏贴快捷键
习惯了用Ctrl+C复制，所以也直接把终端的快捷方式给改了。里面有各种快捷方式，可以根据自己的习惯自定义啊。
![](/images/20190610_4.png)

## 17、几个好玩的命令
### boxes
`sudo apt install boxes`
![](/images/20190610_5.png)
### 终端跑小火车
`sudo apt install sl`
`sl`
![](/images/20190610_6.png)
### 黑客帝国代码雨
`sudo apt install cmatrix`
`cmatrix`
`ctrl+z` 退出
![](/images/20190610_7.png)
### screenfetch
`sudo apt install screenfetch`
`screenfetch`
![](/images/20190610_8.png)
## 18、顶栏自动隐藏
安装Hide Top Bar插件
[https://extensions.gnome.org/extension/545/hide-top-bar/](https://extensions.gnome.org/extension/545/hide-top-bar/)

在全屏显示窗口时就会自动隐藏深色难看的顶栏了

## 19、设置字体缩放比例。
在字体这里设置缩放比例为1.25，看起来就大很多了，接近我在win 10 的大小习惯
![](/images/20190610_9.png)
## 20、安装Mac OS 风格主题
参考
[https://blog.csdn.net/jasonzhoujx/article/details/80400245](https://blog.csdn.net/jasonzhoujx/article/details/80400245)

[到这个网站找各种主题](https://www.opendesktop.org/s/Gnome)
效果如图
![](/images/20190610_10.png)

## 21、修改登录界面的图片
修改这个文件夹
`/etc/alternatives/gdm3.css`
找到这段，修改成自己的图片路径, 以及下面几行css设置.
![](/images/20190610_11.png)

## 22、添加应用启动图标
安装好pycharm后，没有启动的图标，所以自己设置一个。

首先找到图标文件，[链接](https://icon-icons.com/zh/%E5%9B%BE%E6%A0%87/pycharm/93936#256)

然后在`/usr/share/applications/` 目录下添加`Pycharm.desktop`。

修改这个文件。注意，其中的Exec改成执行的命令，Icon改成自己图标的路径。
```
[Desktop Entry]
Type=Application
Name=Pycharm
GenericName=Pycharm3
Comment=Pycharm3:The Python IDE
Exec=bash /home/jamey/programs/pycharm-2019.1.3/bin/pycharm.sh
Icon=/home/jamey/programs/pycharm-2019.1.3/pycharm.ico
Terminal=pycharm
Categories=Pycharm;
```
出现图标
![](/images/20190610_12.png)