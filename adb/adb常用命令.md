#adb常用命令

## 安装与卸载

1. 安装android studio编译的apk包 `adb install -r app/build/output/apk/app-debug.apk`

2. 卸载某一个apk包 `adb uninstall com.star.demo`

## adb shell命令

1. 查看某一个app的activity栈 `adb shell dumpsys activity activities | grep "org.houxg.leamonax"` 
2. 查看当前界面的activity  `adb shell dumpsys activity top`
3. 查看某一个app的service栈 `adb shell dumpsys activity services | grep "tv.shou.android"`
4. 查看手机处理器的信息 `adb shell cat /proc/cpuinfo`
5. 列出手机上已经安装的app的包名 `adb shell "pm list packages"|cut -f 2 -d ":"`
6. 查看出当前的通知栏信息 `adb shell dumpsys notification`


## adb connect
1. 在同一个局域网内，`adb connect 192.168.2.11`可以连接到指定ip的android设备
2. `adb disconnect` 断开连接

## wifi adb
1. wifi环境下连接adb


## 开启关闭adb服务
1. 开启服务 `adb start-server`
2. 关闭服务 `adb kill-server`

## 重启手机
手机卡死了,只要能连上adb `adb reboot` 解决重启问题

## 开发中常用
1. `adb device` 查看当前设备的链接状态
2. 上面介绍的开启关闭adb服务
3. `adb logcat -t` 用来查看应用中某个tag的日志




## adb shell 数据库操作相关
手机adb连接到电脑上，adb shell终端打开数据库操作如下:

先输入`adb shell` 连接到你的设备

输入`cd data/data/com.android.proviers.contacts/databases/`进入到数据库文件所在的目录

输入`sqlite3 contacts2.db` 进入数据库中

输入`select * from table1 where ...`查询表

如果不知道表的名称，可以用 `.tables` 列出当前数据库的所有表

如果不知道某一个表的构成结构，可以用`.schema contacts`（contacts是表的名字）

想要知道更多，可以输入 `.help`

## adb shell get the device properties
获取全部的属性值 `adb shell getprop`
获取某一个属性值 `adb shell getprop ro.build.version.sdk`

## aapt命令获取apk详细信息（包名、版本号、版本名称、兼容api级别、启动Activity等）
aapt dump badging app-debug.apk

## 获取手机上安装的app的versionName,versionCode
adb shell dumpsys package com.star.contacts | grep versionName  //获取versionName

adb shell dumpsys package com.star.contacts | grep versionCode  //获取versionCode

adb shell dumpsys package com.star.contacts //获取详细信息

## 判断apk所采用的签名
apksigner verify --verbose /Users/app_release.apk

## 抓取log
Mac用户推荐用[pidcat](https://github.com/JakeWharton/pidcat/)

pidcat com.star.contacts 抓取log

pidcat com.star.contacts > 1.txt 写入日志文件中

## adb pull命令
当我们需要将手机app里面的私有目录文件拷贝出来（例如db文件），那么就可以通过以下方式

首先进入

adb shell

接着执行

su(切换到root环境，没有做实验，是否必须得是root才行)

接着执行cp命令，将手机应用内的文件复制到sd卡中

cp /data/data/com.xxx.star/tinker/patch-f016f6df/dex/changed_classes.dex.jar /mnt/sdcard

接着我们通过adb pull命令，将sd卡的文件拉取到本地电脑上，比如mac

adb pull sdcard/changed_classes.dex.jar（这个时候拷贝到mac上的是当前终端所在的目录下）

## 查看android手机arm版本
adb shell getprop ro.product.cpu.abi

## 查看android手机dpi
adb shell wm density

## 查看android屏幕分辨率
adb shell wm size

## kill app
adb shell am kill 包名

