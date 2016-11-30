#adb常用命令

##安装与卸载

1. 安装android studio编译的apk包 `adb install -r app/build/output/apk/app-debug.apk`

2. 卸载某一个apk包 `adb uninstall com.star.demo`

##adb shell命令

1. 查看`adb shell dumpsys activity activities | grep "org.houxg.leamonax"`
2. 查看当前界面的activity  `adb shell dumpsys activity top`
