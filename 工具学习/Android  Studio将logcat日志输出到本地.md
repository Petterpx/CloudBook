# Android  Studio将logcat日志输出到本地

Mac



必备条件：安装ADB



在终端使用以下命令记录：

adb logcat  -v time > /(你要保存的位置)

> Adb logcat -v time > /Users/petterp/Documents/logs/log.txt



然后正常Android Studio运行即可；

关闭输出日志使用 control+C即可