---
title: "Darknet  目标检测开源库学习记录"
date: 2019-6-28
draft: false

categories: ["深度学习"]
---



## 1. 效果展示
[官网链接](https://pjreddie.com/darknet/yolo/)

darknet 实现了c语言版本的yolo v3, 不依赖任何其他库. 因此安装非常简单.

效果图:
![](/images/20190628_1.png)
![](/images/20190628_2.png)
![](/images/20190628_3.png)

---

## 2. 安装方法

如何安装?
```bash
git clone https://github.com/pjreddie/darknet
cd darknet
make
```

然后下载yolov3权重, 放到darknet根目录下
`wget https://pjreddie.com/media/files/yolov3.weights`


执行命令`./darknet detector test cfg/coco.data cfg/yolov3.cfg yolov3.weights data/dog.jpg` 测试一下效果, 生成的图片保存在darknet根目录下

---

## 3. 常用命令

以下是几个常用的检测命令
```
# 调用摄像头进行目标检测
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights

# 对视频进行目标检测
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights <video_file>

# 检测单独的一张图片
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg

# 设置阈值的检测
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg -thresh 0

# 对多张图片的检测, 输入命令后, 输入图片路径
./darknet detect cfg/yolov3.cfg yolov3.weights

# 指定摄像头设备, 加参数 -c
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights -c 1
```
---

## 4. GPU加速
用CPU进行测试非常的慢, 下面是官网的描述, 一张图10s(在我的电脑上还不止)
![](/images/20190628_4.png)

使用GPU可以大幅提高速度, 提升有多少呢? 在我的Geforce 940MX 辣鸡显卡上, 都可以实现比较卡顿的摄像头目标检测了! 如果是高性能的显卡, 想必会非常流畅(羡慕)

当然, 用GPU是需要安装好cuda的. 我用的是cuda 10.0

如何开启GPU模式? 修改makefile, GPU=0 改成GPU=1, 然后重新make.
![](/images/20190628_5.png)

---
## 5. 安装opencv
建议安装C++版本的opencv, 安装好了同样是改成OPENCV=1.  (因为装了opencv他才会在屏幕上显示出检测的结果)

[如何在ubuntu 18.04 上安装opencv 3.4.6 ? 看这篇教程, 亲测有效. 除了后期有一丁点的不同基本顺利安装.](https://blog.csdn.net/cocoaqin/article/details/78163171)

---
## 6. 几点小提示

查看`./cfg/yolov3.cfg` 文件, 有两种模式, 根据自己的实际需要进行注释. 比如说我们做测试, 就改成testing模式.
![](/images/20190628_6.png)
如果GPU太垃圾(比如我), 一测试就报显存溢出错误, 可以设置下面的width 和 height, 将它设小一点就可以了. 
![](/images/20190628_7.png)

---
## 7. 使用网络摄像头(手机)之一

使用网络摄像头, 用手机作为外接摄像头而不只是笔记本的摄像头来做目标检测, 下面是效果图.

![](/images/20190628_8.png)
我使用的是Droidcam ip摄像头

手机需要下载Droidcam app, 可以直接在google play上下载

[官网: http://www.dev47apps.com/](http://www.dev47apps.com/)

[以及一篇非常好的安装教程](https://www.freeyourdesktop.com/2018/10/need-a-webcam-in-ubuntu-no-problem-thanks-to-android/)

ubuntu 18.04 (linux)下的安装方法是
复制下面这段代码做一个bash脚本, 然后执行它.
```bash
cd /tmp/
sudo apt-get install linux-headers-`uname -r`
bits=`getconf LONG_BIT`
wget https://www.dev47apps.com/files/600/droidcam-${bits}bit.tar.bz2
[[ ${bits} -eq 32 ]] && checksum=90cd43b4745c51cffedc352090912eb1
[[ ${bits} -eq 64 ]] && checksum=9507c0b738f427c5f1dde7b2a364fdfb
echo "${checksum}  droidcam-${bits}bit.tar.bz2" | md5sum -c --
# OK?
tar xjf droidcam-${bits}bit.tar.bz2
cd droidcam-${bits}bit/
sudo ./install
```

建立启动器
`gedit ~/.local/share/applications/droidcam.desktop`
拷贝下面的代码进去
```
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Name=DroidCam
Exec=droidcam
Comment=Use your Android phone as a wireless webcam or an IP Cam!
Icon=droidcam
Categories=GNOME;GTK;Video;
Name[it]=droidcam
```
就可以找到图形程序的图标

然后就可以使用wifi连接手机上的摄像头. 也可以用usb连接的方式.
![](/images/20190628_9.png)

---
用命令`ls /dev/video*` 查看电脑上的摄像头设备. 目前有两个. 这个video1就是ip摄像头了.
![](/images/20190628_10.png)
在使用darknet的时候, 在后面加参数`-c 1` 就可以指定摄像头设备了.

比如`./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights -c 1`

---
## 8. 使用网络摄像头(手机)之二
但如果用ip摄像头的话, 一来比较卡顿, 二来如果在外面没有wifi怎么办呢? 那么用usb就是个更好的选择. 参考这篇文章弄好了usb摄像头
[https://xpenxpen.iteye.com/blog/2182397](https://xpenxpen.iteye.com/blog/2182397)

安装adb `sudo apt-get install adb`

查看设备`adb devices`

输入`adb forward tcp:4747 tcp:4747  ` 启用摄像头 (改成自己的端口)

手机打开开发者模式, 进开发者选项把usb调试打开

客户端开启usb模式

![](/images/20190628_11.png)

---
## 9. 保存检测视频到本地

darknet官方似乎并没有一个简单的参数可以将检测的视频保存在本地, 找了好多文章后终于找到一个靠谱的方法. 

[来源链接](https://github.com/pjreddie/darknet/issues/1235)

![](/images/20190628_12.png)

思路是, darknet提供了一种参数, -prefix, 可以将视频检测的结果输出为一系列的图片. 我们将这些图片保存在tmp文件夹, 然后使用视频转化工具ffmpeg 将这一系列的图片转化为视频.

下面是修改了的bash脚本, 保存为run.sh

```py
#!/bin/bash

./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights $1 -prefix ./tmp/pictures
ffmpeg -i ./tmp/pictures_%08d.jpg $2

rm ./tmp/pictures_*.jpg
```
我稍微修改了github issue上的脚本, 使得我们可以指定输入和输出的文件. 
执行这条命令就可以保存视频了
```bash
./run.sh ./data/test-5.mp4 ./outputfiles/test-5.mp4
```

调用摄像头然后保存的话, 也稍微修改一下脚本就好了

---

perfect
