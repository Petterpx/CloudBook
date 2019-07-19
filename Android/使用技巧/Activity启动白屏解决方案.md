# Activity启动白屏解决方案

[参考链接：](https://www.jianshu.com/p/7469704e3ba0)

一些相应的方法：

```java
先学会常用的Theme主题功能：

Activity显示为对话框模式
android:theme="@android:style/Theme.Dialog"
不显示应用程序标题栏
android:theme="@android:style/Theme.NoTitleBar"     
不显示应用程序标题栏，并全屏.
android:theme="@android:style/Theme.NoTitleBar.Fullscreen"     
背景为白色
android:theme="Theme.Light "     
白色背景并无标题栏
android:theme="Theme.Light.NoTitleBar"     
白色背景，无标题栏，全屏
android:theme="Theme.Light.NoTitleBar.Fullscreen"     
背景黑色
android:theme="Theme.Black"     
黑色背景并无标题栏
android:theme="Theme.Black.NoTitleBar"     
黑色背景，无标题栏，全屏
android:theme="Theme.Black.NoTitleBar.Fullscreen"     
用系统桌面为应用程序背景
android:theme="Theme.Wallpaper"     
用系统桌面为应用程序背景，且无标题栏
android:theme="Theme.Wallpaper.NoTitleBar"     
用系统桌面为应用程序背景，无标题栏，全屏
android:theme="Theme.Wallpaper.NoTitleBar.Fullscreen"     
透明背景
android:theme="Theme.Translucent"    
透明背景并无标题
android:theme="Theme.Translucent.NoTitleBar"     
透明背景并无标题，全屏
android:theme="Theme.Translucent.NoTitleBar.Fullscreen"     
面板风格显示
android:theme="Theme.Panel "
```



第一种方式：透明窗口

```
<style name="TransluteTheme" parent="AppTheme">
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:screenOrientation">portrait</item>
  </style>
```

![Jietu20190719-155340-HD](http://ww3.sinaimg.cn/large/006tNc79ly1g557am2jjig307v0gou0z.gif)



第二种(主流方式，给启动加张背景图片)

```java
<style name="SplashTheme" parent="AppTheme">
    <item name="android:windowBackground">@android:color/holo_blue_dark</item>
    <item name="android:windowFullscreen">true</item>
    <item name="android:windowContentOverlay">@null</item>
</style>
```

![Jietu20190719-160236-HD](http://ww3.sinaimg.cn/large/006tNc79ly1g557kqpisqg307v0gohdx.gif)