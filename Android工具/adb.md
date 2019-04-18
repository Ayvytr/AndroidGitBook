# ADB和ADB idea



```
查看前台Activity adb shell dumpsys activity activities | grep mFocusedActivity

adb version 查看版本

adb devices 查看连接设备

adb install <apkfile> 安装

adb uninstall <packge> 卸载

adb start-server 启动Server

adb kill-server 停止Server

adb logcat 查看日志

adb get-serialno 获取序列号

adb shell pm list packages 列出所有包名

adb shell pm list packages -s 列出系统包名

adb shell pm list packages -3 列出第三方包名

adb shell pm list packages | grep qq grep过滤

adb shell pm clear <packagename> 清除应用数据

adb shell cat /sys/class/net/wlan0/address 获取MAC地址

adb shell getprop ro.product.model 查看设备型号

adb shell getprop ro.build.version.release 查看Android版本

adb shell wm size 查看分辨率

adb shell wm density 查看屏幕密度

无线连接，需要借助USB线
adb tcpip 8888
adb connect ip:8888
adb disconnect ip:8888


查看应用列表：
adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]
查看所有应用 adb shell pm list packages

```





```
安装APK

adb install [-lrtsdg] <path_to_apk>
adb install 后面可以跟一些可选参数来控制安装 APK 的行为，可用参数及含义如下：

参数	含义
-l	将应用安装到保护目录 /mnt/asec
-r	允许覆盖安装
-t	允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用
-s	将应用安装到 sdcard
-d	允许降级覆盖安装
-g	授予所有运行时权限

```

```
卸载apk adb uninstall [-k] <packagename>
<packagename> 表示应用的包名，-k 参数可选，表示卸载应用但保留数据和缓存目录。 

```





