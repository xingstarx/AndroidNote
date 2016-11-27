#adb常用命令

adb install -r app/build/output/apk/app-debug.apk

adb uninstall com.star.demo

adb shell dumpsys activity activities | grep "org.houxg.leamonax"

