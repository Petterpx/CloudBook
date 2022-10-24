# 写一个适配 Android12-exported 的小插件

## 📚 背景

从 `Android12` 开始，如果我们的 `tagSdk` >=31, 即以 `Android 12` 或更高版本为目标平台时，且包含使用 [intent 过滤器](https://developer.android.com/guide/components/intents-filters#Receiving)的 [activity](https://developer.android.com/guide/components/activities/intro-activities)、[服务](https://developer.android.com/guide/components/services)或[广播接收器](https://developer.android.com/guide/components/broadcasts)，则必须为这些应用组件显式声明 [`android:exported`](https://developer.android.com/guide/topics/manifest/activity-element#exported) 属性。

如果你满足上述条件，并且 **tagSdk>=31** ，而未声明 `exported` 属性，则在不同的 Agp 版本有着以下不同提醒方式：

- `Agp7.0` 及以上，在 build 时会出现下面的报错：

  > Manifest merger failed : android:exported needs to be explicitly specified for <activity>. Apps targeting Android 12 and higher are required to specify an explicit value for `android:exported` when the corresponding component has an intent filter defined. See https://developer.android.com/guide/topics/manifest/activity-element#exported for details

- **Agp7.0以下**

  > 则并不会在编译时报错，而是在安装后打开相关页面时报错，相应的，**Android Studio** 会以 ⚠️ 的样式提醒你添加 `exported` 。

恰好最近也正好在做相关的适配，于是就查了下，发现了恋猫的小郭大佬写的这样一篇文章，[Android12的适配](https://juejin.cn/post/7037105000480243748#heading-12)，其中关于 `exported` 部分，比较简单实用，但相对来说，存在版本差异，真实使用起来还需要再改一下。本文就是针对其做的一个完善，并将其抽离成了一个小插件，以便更好的复用。

## 💬 插件简介

[manifest-exported-plugin](https://github.com/xiachufang/manifest-exported-plugin)

一个用于快速适配 [Manifest-exported](https://github.com/xiachufang/manifest-exported-plugin) 的小插件,通过修改 `manifest` 文件，从而做到适配。

## 👨‍💻‍ 使用方式

### 1. 添加jitpack

**build.gradle**

Gradle7.0 以下

```groovy
buildscript {
	repositories {
			// ...
			maven { url 'https://jitpack.io' }
	}
}
```

`Gradle7.0+` ,并且已经对依赖方式进行过调整，则可能需要添加到如下位置：

> **settings.gradle**
>
> ```groovy
> pluginManagement {
>  repositories {
>      //...
>         maven { url 'https://jitpack.io' }
>     }
> }
> ```

#### Gradle

```groovy
dependencies {
      classpath 'com.github.xiachufang:manifest-exported-plugin:1.0.6'
}
```

### 2. 添加插件

在主app Model中添加：

```groovy
apply plugin: 'com.xiachufang.manifest.exported'
```

> 或
>
> ```
> plugins {
>  id 'com.xiachufang.manifest.exported'
> }
> ```

### 3. 参数说明

app-build.gradle

```groovy
apply plugin: 'com.xiachufang.manifest.exported'
...
  
exported {
    actionRules = ["android.intent.action.MAIN"]
    enableMainManifest false
    logOutPath "自定义的日志输出目录,如果不存在会自动创建"
}
```

- **logOutPath** 日志输出目录，默认 **app/build/exported/outManifest.md**

- **actionRules** action的匹配项(数组), 如：

  ```
  <activity android:name=".simple.MainActivity" >
        <intent-filter>
            // action 对应的 android:name 可与actionRules 数组任意一项匹配 ,并且当前没有配置exported
              // -> yes: android:exported="true"
              // -> no: android:exported="false"
          <action android:name="android.intent.action.MAIN"/>
          <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
  </activity>
  ```

- **enableMainManifest** 是否对主 `model-AndroidManifest` 进行修改

  对于 `主model` ,属于业务可控的，建议开发者自行调整。

  插件默认不会对主 `model-AndroidManifest` 进行修改,如果发现可用匹配上述规则的，即会进行修正。
  
  开发者可根据日志中的提示，进行修改。


## 📰 相关截图说明

默认情况下插件的输出目录如下所示，主 model/build/exportred/outManifestLog.md,默认日志如下：

![image-20220627110207568](https://tva1.sinaimg.cn/large/e6c9d24ely1h3mmdgxn8mj229j0u011e.jpg)

## 💭 注意事项

对于主model下的 `manifest` ，默认不进行适配(可开关 **enableManifest** )，会通过日志进行输出，建议大家自行对比调整。

> 为什么默认不对主 `model` 进行适配？
>
> - 对于业务 `model` ,我们建议开发者自行适配,这属于我们可控范围，适配来说主要就是为了不可控的，即第三方 `aar`
> - 修改之后，会影响原有的 `manifest` 代码风格，需要重新格式化一下，相比默认的，增加了不少空格，暂时不知道怎么解决。

## 🤔 原理解析

> 通过插入到 processxxxMainManifest Task之前，提前对manifest进行修改。

**为什么不插入到 processxxxManifest ,这样不就不用操作 manifest 源文件了吗，只需要对build下的 mergeManifest 进行操作不就行了？**

首先说明这两者的顺序：

1. processxxxMainManifest
2. processxxxManifest

我们都知道，**build** 时会将所有 `aar` 里的 `manifest` 的全部进行合并，如果有异常会进行报错。

但在不同的 **AGP** 版本，**它们的检测时机不一样**。

通常情况下，在 `processxxxMainManifest` 结束后，我们就可以拿到已经合并好的 `manifest` 文件，此时就可以直接进行更改适配。

在agp7.0这个思路没有问题，因为 `processxxxMainManifest` 里面不会去检测 `manifest` 是否合并成功，而会在 `processxxxManifest` 去检测。

但在agp7.0以上，因为会先去检测 manifest 是否合并成功，这就导致我们后续的任务没法正常执行，所以我们没有办法将任务插入到 `processxxxMainManifest` 之后，只能在其之前执行，所以此时我们只能去修改 manifest 源文件从而做到适配。

具体的适配上也比较简单，我们使用 XmlParser 拿到manifest里的节点，通过如下几个条件层层判断，最终完成适配：

1. 判断是不是 `activity`、`services`、`broadcasts` 以及是不是包含了 <intent-inlfter>
2. 判断 `exported` 是否为 `null`
3. 判断 `action` 是否匹配我们的条件

> 需要注意的是，在 `Agp4.2` && `gradle6.7` 及以下时，`tagSdk31`，就算你不去写 **exported** ,编译也不会报错，但此时运行在Android12 手机上时，就会出现相应的报错提示。所以在写插件时我们需要对这种情况进行容错,对于未适配的 `主mainManifest` ，进行报错以便提醒用户。

## 

## 🔖 感谢

- [Android 12 快速适配要点](https://juejin.cn/post/7037105000480243748)
- [Android Gradle 中的实例之动态修改AndroidManifest文件](https://blog.csdn.net/nihaomabmt/article/details/120285452)
- [封面插图来源-cafonsmota/Carlos Mota](https://cafonsomota.medium.com/android-12-dont-forget-to-set-android-exported-on-your-activities-services-and-receivers-3bee33f37beb)