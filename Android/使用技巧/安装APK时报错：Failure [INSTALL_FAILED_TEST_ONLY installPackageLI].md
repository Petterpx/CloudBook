# 安装APK时报错：Failure [INSTALL_FAILED_TEST_ONLY: installPackageLI]


使用AS自动运行时会在app\build\outputs\apk\debug文件夹下自动生成测试APK：app-debug.apk，

用命令adb install app-debug.apk时报错：Failure [INSTALL_FAILED_TEST_ONLY: installPackageLI]

解决办法：

1. 添加-t参数: 输入命令adb install -t app-debug.apk

2. 在gradle.properties(项目根目录或者gradle全局配置目录 ~/.gradle/)文件中添加：

android.injected.testOnly=false
产生原因：

Android Studio 3.0会在debug apk的manifest文件application标签里自动添加 android:testOnly="true"属性

