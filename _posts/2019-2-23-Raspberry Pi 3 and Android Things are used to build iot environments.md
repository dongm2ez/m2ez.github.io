---
layout: post
title: 用 Raspberry Pi 3（树莓派）和 Android Things 搭建物联网开发环境
categories: development
tags: [Raspberry,物联网]
---

简介树莓派是一个极小的单块电路板，但是却有着计算机的所有基本功能，而 Android Things 是 Google 在 Android 的基础上去掉了一些物联网不需要的库，又加入了一些物联网需要的库而开发的物联网专用操作系统。

Android Things 的宣传语是 「If you can build an app, you can build a device」，翻译过来就是：如果你能构建一个应用，你就能构建一个设备。它的目的就是将原来只有电子工程师或者专门学习的硬件底层进行封装，让软件工程师只需要很少的学习就可以去开发硬件。使得 IoT（物联网）的开发更简单。需要设备

既然是开发物联网，那我们必须有开发物联网的设备，最小成本的设备当属树莓派了，当然你要是能拿到谷歌推荐的那块板子也更好，我目前是使用树莓派来学习的。
以下是我们需要的设备

* 树莓派主板
* 一张8G以上的 Micro SD 卡
* 读卡器
* 树莓派电源线
* HDMI 连接线
* 显示屏一块
* 网线（可选）
* 鼠标（可选）

## 安装 Android Things

安装 Android Things 需要先下载最新的 Android Things 镜像，可以去[https://partner.android.com/things/console/](https://partner.android.com/things/console/)用你的 google 账号登录，新建一个项目，并且构建一个镜像进行下载。 最新的镜像必须有一个项目才可以下载。

下载完成后我们可以用两种方式进行安装镜像，一种是用官方提供的控制台程序另一种是自己烧录。

![](http://img.m2ez.com/image/1.jpg)
￼
工具在这里下载，下载工具后选择自己相应的系统的工具根据提示做就行了。
自己烧录的话以下是教程：

* [Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)
* [Mac](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)
* [Windows](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)

烧录完成后，我们就可以把 sd 卡插入树莓派，连接电源和显示器，如果你烧录的正确，你就会看到显示器上出现 Android Things 的启动画面。
￼
![](http://img.m2ez.com/image/2.jpg)

通过无线和网线都可以将你的电脑和树莓派进行通信，和电脑连通的树莓派被视为一个安卓设备，因此可以使用android开发工具包中的adb进行远程调试。adb 工具可以通过 ip:5555 端口与 android thins开发板进行通讯，如果有鼠标显示器可以直接连接到树莓派上进行网络连接。

```shell
# 先通过有线连接，获取IP
$ ./adb connect <ip-address>
connected to <ip-address>:5555

# adb连接上之后配置 Wi-Fi
$ ./adb shell am startservice \
-n com.google.wifisetup/.WifiSetupService \
-a WifiSetupService.Connect \
-e ssid <Network_SSID> \
-e passphrase <Network_Passcode>

# 验证是否 Wi-Fi 是否连接成功
$ ./adb logcat -d | grep Wifi
...
V WifiWatcher: Network state changed to CONNECTED
V WifiWatcher: SSID changed: ...
I WifiConfigurator: Successfully connected to ...


# 重启，撤掉网线，获取 Wi-Fi 连接后的设备 IP，可通过 HDMI 显示器获得或从路由器后台获得
$ ./adb connect <wifi-ip-address>
connected to <wifi-ip-address>:5555

# 查看设备是否attached
$ ./adb devices
List of devices attached
<wifi-ip-address>:5555 device
```

这时官方的demo我们就可以通过adb的方式发送的树莓派上进行运行了。
官方提供的demo 目录链接 ：[https://developer.android.com/things/sdk/samples-drivers.html](https://developer.android.com/things/sdk/samples-drivers.html)