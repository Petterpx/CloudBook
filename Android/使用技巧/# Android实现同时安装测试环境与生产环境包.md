# ***Android***实现同时安装测试环境与生产环境包

众所周知，相同包名的APP，是不能同时安装的，但是我们实际开发中，测试同学往往在测试环境没问题，上了生产环境，却发现了bug,这时候就只能卸载生产环境的包，再去安装测试环境。如果没有开发流程中缺少自动化打包或者测试同学不保存蒲公英二维码，这时候就会产生多余时间成本。那么有没有一种可能，同时安装测试与生产环境的包呢？

这个当然是可以的，我们更换包名就行了，Android Studio早已为我们准备了相应的操作：

很简单，就一句：

给你的app, buildTypes -debug下面增添

```java
applicationIdSuffix ".debug"
```

相当于在打包时，会为debug的包原包名后增加 .debug.



详细如下：

```java
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            debuggable true
            zipAlignEnabled true  //是否支持zip
            resValue("string", "ENVIRONMENT", "RELEASE")
            buildConfigField "boolean", "LOG_DEBUG", "false"
            resValue "string", "name", "release"
            resValue "int", "logo", "release"
            signingConfig signingConfigs.release

        }
        debug {
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            minifyEnabled false
            zipAlignEnabled true
            debuggable true

            applicationIdSuffix ".debug"
            resValue("string", "ENVIRONMENT", "DEBUG")
            resValue "string", "name", "debug"
        }

    }
```





以上操作适用于大部分同学，但如果你的APP中含有 ContentProvider或者FileProvider(Android7.0文件适配必备)，也就是和包名相关的；或者你想更直接点，直接区分测试与生产的app名及图标，那么你可能需要如下操作了：

#### APP含有ContentProvider

##### 实际场景：***华为推送***

```java
<provider
            android:name="com.huawei.hms.update.provider.UpdateProvider"
            android:authorities="com.test.app.hms.update.provider"
            android:exported="false"
            android:grantUriPermissions="true" /> <!-- 应用下载服务 -->
```

当我们的项目中包含华为push时，往往会有如上代码，此时如果不处理包名，就会出现同时只能安装一个APP，否则adb就会提示 **com.huawei.hms.update.provider.UpdateProvider** 已存在(使用adb命令，adb install ...apk 这样会详细提示)。

##### 处理方式：

```java
       <provider
            android:name="com.huawei.hms.update.provider.UpdateProvider"
            android:authorities="${applicationId}.hms.update.provider"
            android:exported="false"
            android:grantUriPermissions="true" /> 
```



#### APP含有FileProvider

Android7.0文件适配时常见场景

```java
<provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
```

适配也很简单，将 authorities=""  里的包名改为 ***${applicationId}*** 即可



#### 动态替换app名，图标

都到这一步了，那不如更友好点，让测试同学更好辨认

app,src下创建如下文件夹

![image-20200308235349364](https://tva1.sinaimg.cn/large/00831rSTly1gcmyj6gg3yj30ey0psaby.jpg)

为了区分，我这里创建了debug,和release两个文件夹，内部分别创建了mipmap,和values文件夹，类似于main文件夹下的src。当然debug和release的名字可以自己定。

***Strings.xml***

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="name">测试</string>
</resources>
```





接着打开app的build文件，修改如下 buildType的代码：

```java
release {
	...
    //位置在这里
    resValue "int", "logo", "release"
    manifestPlaceholders = [icon: "@mipmap/logo"]
    ...
}
debug {
  ....
      
    //位置在这里
    resValue "string", "name", "debug"
    manifestPlaceholders = [icon: "@mipmap/logo"]
  ...
}
```

然后修改AndroidManifest如下

![image-20200309000103284](https://tva1.sinaimg.cn/large/00831rSTly1gcmyqp8qmfj310q0hk0wo.jpg)



效果如下：

![image-20200309000332542](https://tva1.sinaimg.cn/large/00831rSTly1gcmytabkuoj30cs08g79k.jpg)

好了，实现了，还是挺简单,是不是很直观了。





### 叮叮当

***需要注意的地方***

> 如果你的APP内含有分享或者推送，那么测试版如果与线上用的是同一个appid与servert,那么测试版可能都会失败，当然这也很正常(如果不是同一个，自己处理下即可，怎么处理呢，楼下截图)。所以这点需要注意，逻辑上的功能都没什么问题。

##### 如果不是同一个id，处理方式如下

同样是buildType下更改debug和release,分别对应的不同id,

```
  ...
  buildConfigField "String", "BaseUrl", "\"https://www.google.com/\""
	buildConfigField "String", "webUrl", "\"https://www.google.com/""
   buildConfigField "String", "wxAppid", "aasdadasdadadasda"
```

使用时如下：

```java
class Test {
    fun test() {
        BuildConfig.webUrl
        BuildConfig.BaseUrl
    }
}
```

这样在打包时相应的id就会自动对应。省的自己判断了。



当然这都是很基础的一些东西，如果有帮到你的地方，不胜荣幸哈。