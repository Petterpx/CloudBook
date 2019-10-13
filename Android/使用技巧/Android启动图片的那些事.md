# APP启动图片的那些事



### 一个自定义主题模板：

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
    <item name="android:windowBackground">@mipmap/launcher_app</item>
    <item name="textAllCaps">false</item>
    <!--用于解决底部导航栏遮挡窗口-->
    <item name="android:windowDrawsSystemBarBackgrounds" tools:ignore="NewApi">false</item>
</style>
```





### 如何添加app启动图片

```xml
<item name="android:windowBackground">@mipmap/launcher_app</item>
```



### 添加了启动图片，我的Activity中多渲染了一层背景

在你的初始化Activity中，onCreate方法中，setConteVIew之前加上如下代码：

即设置背景色为纯白色，注意必须在 setConteView之前.

```java
this.getWindow().getDecorView().setBackgroundColor(Color.WHITE);
```



### app启动时底部导航栏遮挡app启动图片解决方案；

在主题中加入以下代码

```xml
<item name="android:windowDrawsSystemBarBackgrounds" tools:ignore="NewApi">false</item>
```

例如：

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
 	。。。
    <!--用于解决底部导航栏遮挡窗口-->
    <item name="android:windowDrawsSystemBarBackgrounds" tools:ignore="NewApi">false</item>
</style>
```

##### 相关解释：

> 翻译： 
>  标志是指示此窗口是否负责绘制系统栏的背景。如果真正的窗口不浮，系统栏被画在这个窗口透明背景和相应领域内statusbarcolor和navigationbarcolor指定的颜色。对应于flag_draws_system_bar_backgrounds。 
>
>  可以看出该属性是负责绘制系统栏的背景的，如果真正的窗口被遮盖了，设置true，则会绘制系统栏的背景，使真正的窗口上移，不被遮挡住。 
>
> 注意：最低适配版本 API:21





