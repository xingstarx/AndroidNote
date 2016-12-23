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
