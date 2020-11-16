\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.jianshu.com\](https://www.jianshu.com/p/0b25b8531a3c)

搭建编译环境，并安装相关关联库
===============

```
\[compiler\] sudo apt-get install build-essential
\[required\] sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
\[optional\] sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

下载 opencv 并且编译
==============

```
git clone https://github.com/opencv/opencv.git
cd opencv
mkdir release
cd release
cmake -D CMAKE\_BUILD\_TYPE=RELEASE -D CMAKE\_INSTALL\_PREFIX=/usr/local ..
sudo make
sudo make install
sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'  
sudo ldconfig
```

测试安装
====

切换到 samples 目录下面，并编译 samples 程序

```
cd ../samples/  
sudo cmake .  
sudo make -j $(nproc)
```

然后切换到 cpp 目录下运行测试程序

```
cd cpp/  
./cpp-example-facedetect XX (图片路径)
```

显示图像，测试成功