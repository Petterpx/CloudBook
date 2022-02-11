# 哪怕不学Gradle,这些开发常见操作，你也值得掌握

![gradle](https://tva1.sinaimg.cn/large/008i3skNly1gyhnz8rkrfj30l70b874j.jpg)

> `Gradle` 是每个 `Android` 同学都逃不开的一个话题。
>
> 你是否看到别人的 `Gradle` 文件干净又卫生？而自己的又是一团乱麻🏷
>
> 不用怕,本篇将结合我的开发日常，将一些常用的操作分享出来，希望可以帮到像我一样不怎么会[玩]`Gradle` 的同学，相信会对大家有所帮助。

## 模板代码提取



这是最基础的操作了，对于一个普通 `model.gradle` ,默认的配置如下：

![image-20220117094502841](https://tva1.sinaimg.cn/large/008i3skNly1gygfdlsvipj31a90u076h.jpg)

如果我们每个 `model` 都这样写，那岂不是很麻烦，那么让我们提取通用代码：

#### 优化步骤

新建一个 `gradle` 文件，命名为 `xxx.gradle` ,复制上述 `model` 里的配置，放到你的项目中,可以自定义修改一些通用内容，在其他`model` 中依赖即可,如下所示：

**这是一个播放器model**

```groovy
// 这就是刚才新建的默认gradle文件,
// 注意：如果你的default.gradle是在项目目录下，请使用../,如果仅在app下，请使用./
apply from: "../default.gradle"
import xxx.*

android {
  	// 用于隔离不同model的资源文件
    resourcePrefix "lc_play_"
}


dependencies {
    compileOnly project(path: ':common')
    api xxx
}
```

> 上述的 **android{}** , **dependencies{}** 
>
> 其内部的内容都会在 `default.gradle` 的基础上叠加，对于唯一的键值对，会进行替换。

## 定义统一的config配置

在项目中，你是如何去写你的版本号等其他默认配置呢？

![image-20220113100724957](https://tva1.sinaimg.cn/large/008i3skNly1gybtjmut15j31bg0jqabo.jpg)

对于一个新项目，其默认的配置如下所示，每次新创建 `model` ,也需要定义其默认参数，如果每次都直接在这里去改动，那么如果版本变化，意味着我们需要修改多次，这并不是我们想看到的效果。

#### 优化步骤

新建 **config.gradle** ，内容如下：

```groovy
// 一些配置文件的保存

// 使用git的commit记录当做versionCode
static def gitVersionCode() {
    def cmd = 'git rev-list HEAD --count'
    return cmd.execute().text.trim().toInteger()
}

static def releaseBuildTime() {
    return new Date().format("yyyy.MM.dd", TimeZone.getTimeZone("UTC"))
}

ext {
    android = [compileSdkVersion: 30,
               applicationId    : "com.xxx.xxx",
               minSdkVersion    : 21,
               targetSdkVersion : 30,
               buildToolsVersion: "30.0.2",
               buildTime        : releaseBuildTime(),
               versionCode      : gitVersionCode(),
               versionName      : "1.x.x"]
}
```

使用时：

```groovy
android {
    def android = rootProject.ext.android
    defaultConfig {
        multiDexEnabled true
        minSdk android.minSdkVersion
        compileSdk android.compileSdkVersion
        targetSdk android.targetSdkVersion
        versionCode android.versionCode
        versionName android.versionName
    }
 }
```

## 配置你的build

### 配置不同build类型

在开发中，我们一般会有多个环境，比如 **开发环境** ，**测试环境**，**线上环境**：

```groovy
buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }

    dev{
        // initWith代表的是允许从其他build类型进行复制操作，然后配置我们想更改的设置
      	// 这里代表的是从release复制build配置
        initWith release
        // 清单占位符
        manifestPlaceholders = [hostName:"com.petterp.testgradle.dev"]
        // 会在你原包名后面新增.test
        applicationIdSuffix ".dev"
    }
}
```

如上所述，`dev` 是我们新增的 **build类型** ,当新增之后，我们就可以在命令行使用如下匹配的指令，或者点击As 最右侧，gradle图标，选择app(根据自己build的配置位置而定，一般默认是app-model)，选择other,即可看到多了如下几个指令：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyepjrwhvpj312c0u076r.jpg" alt="image-20220114095904490" style="zoom: 33%;" />

当然你也可以选择如下命令行执行,以便在 **Jenkins** 或者 **CI** 下 `build` 时执行：

```groovy
gradlew buildDev
gradlew assembleDev
```

> 注意，mac下是gradlew开头，windows下可能是./gradlew

---

### 配置变体

对于开发中，我们一般都有多渠道的需求，一般而言，如果仅仅是多渠道我们可以选择使用第三方 [walle](https://github.com/Meituan-Dianping/walle) 去做，如果我们可能还有更精细的设置，比如针对这个 **build类型**，我们很可能对应了不同的默认配置等，比如配置不同的 `applicationId` ,资源。

如下所示：

```
// 变体风味名，如果只设置一个，则所有变体会自动使用，如果存在两个及以上，需要在变体中指定,并且变体需要与分组匹配。
// 风味名，类似于风格，分组的意思。
flavorDimensions "channel"
// flavorDimensions ("channel","api")
productFlavors {
    demo1 {
    		// 每一个变体都必须存在一个风味，默认使用flavorDimensions(仅限其为单个时)的值，否则如果没提供，则会报错。
        dimension "channel"
        // appid后缀,会覆盖了我们build类型中的applicationIdSuffix
        applicationIdSuffix ".demo"
        // 版本后缀
        versionNameSuffix "-demo"
    }
    demo2 {
        dimension "channel"
        applicationIdSuffix ".demo2"
        versionNameSuffix "-demo2"
    }
}
```

然后查看我们的 **build Variants**:

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyepjsdbhpj30p20cgjry.jpg" alt="image-20220115110605931" style="zoom:50%;" />

**Gradle** 会根据我们的 `变体` 和 `build类型` 自动创建多个build变种，按照 `变体名-build类型名` 方式命名。

> 在配置变体时，我们也可以替换在 `build类型` 中设置的所有默认值，具体原因是，在添加 `build类型` 时，默认的 `defaultConfig` 配置其实是属于 `ProductFlavors` 类，所以我们也可以在任意变体中替换所有默认值。

---

### 组合多个变体

在某些场景下，我们可能想将多个产品的变体组合在一起，比如我们想增加一个 **api30** 的变体，并且针对这个变体，我们想**让demo1和demo2与分别也能与其组合在一起** ，即也就是当channel是demo1时api30下对应的包。

理解起来有些拗口，示例如下所示，我们更改上面的配置:

```groovy
  flavorDimensions("channel", "api")
    productFlavors {
        demo1 {
            dimension "channel"
            applicationIdSuffix ".demo"
            versionNameSuffix "-demo"
        }
        demo2 {
            dimension "channel"
            applicationIdSuffix ".demo2"
            versionNameSuffix "-demo2"
        }
        minApi23 {
            dimension "api"
            minSdk 23
            applicationIdSuffix ".minapi23"
            versionNameSuffix "-minapi23"
        }
    }
```

最终如下所示,左侧是 `gralde` 生成的 **build变种** ，右侧对应其中 `demo1MinApi23Debug` 打包后的产物具体信息：

![image-20220115114123883](https://tva1.sinaimg.cn/large/008i3skNly1gyepjpks9oj323i0motda.jpg)

**所以我们可以总结为：**

最终我们在打包时，我们的包名和版本名会根据多个变体混合生成，具体如上图所示，然后分别使用了两者都具有的配置，当配置出现重复时，优先以开头的变体配置作为基准。

**比如如果我们给demo1变体也配置了最低sdk版本是21,那么最终打出来的包minSdk也会是21，而不是minApi23中的minSdk配置，这点需要注意。**

---

#### 解疑

那么 `变体` 和 `build类型` 两者到底应该怎么选？似乎两者好像很是相似？

其实不难理解，如下所示：

比如你新增了一个变体 `firDev` ,那么默认情况下就会有如下的 **build命令** 生成

```
firDevDebug
firDevRelase
firDevXXX(xxx是你自定义的build类型)
```

> 需要注意的是 `debug` 和 `relase` 是默认就会存在的，我们可以选择覆盖，否则就算移除，其也会选择默认设置存在

即也就是最终 `gradle` 会帮我们每个变体都生成相应的 **build类型** 对应的命令，变体就相当于不同的渠道，而 **build类型** 就相当于针对这个渠道，存在着多种环境，比如 `debug`,`relase`,你自定义的更多build类型。

- 所以如果你的场景仅仅是想对应几个不同环境，那么直接配置 **build类型** 即可；
- 如果你可能希望区分不同的包下的依赖项或者资源配置，那么配置变体即可。

---

### 过滤变体

`Gradle` 会为我们配置的 **所有变体** 和 **build类型** 每一种可能组合都创建一个 `build变种` 。当然有些变种，我们并不需要，所以我们可以在相应模块的 `build.gradle` 中创建 **变体过滤器** ，以便移除某些不需要的变体配置。

```groovy
android{
	...
	variantFilter { variant ->
        def names = variant.flavors*.name
        if (names.contains("demo2")) {
            setIgnore(true)
        }
    }
	...
}
```

效果如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyepjtb49xj31ho0j4tb7.jpg" alt="image-20220115120754881" style="zoom:50%;" />

---

### 针对变体配置依赖项

我们也可以针对上面这些变体，进行不同的依赖。比如：

```groovy
 demo1Implementation  xxx
 minApi23Implementation xxxx
```

## 常见技巧

### 关于依赖管理

对于一些环境下，我们并不想在线上依赖某些库或者 **model** ，如果是三方库，一般都会有 **relase** 下依赖的版本。

> 如果是本地model,目前已经引用到了，所以就需要对于线上环境做null包处理，只留有相应的包名与入口，具体的实现都为null.
>

#### 限制依赖条件为build类型

```groovy
debugImplementation project(":dev")
releaseImplementation project(":dev_noop")
```

有一点需要注意，当我们使用默认的 `debugImplementation` 和 `releaseImplementation` 进行依赖时,最终打包时是否会依赖其中，取决于我们 **使用的build命令中build类型是不是debug或者relase** ,如果使用的是自定义的 `dev` ,那么上述的两个 model 也都不会依赖，很好理解。

#### 限制依赖条件为变体

相应的，如果我们希望当前的依赖的库或者model 不受 **build类型** 限制，仅受 **变体** 限制，我们也可以使用我们的 `变体-Implementation` 进行依赖，如下所示：

```groovy
demo1Implementation project(":dev")
```

这个意思是，**如果我们打包时使用demo1相应的gradle命令，比如assembleDemo1Debug,那么无论当前build类型是debug还是release或者其他，其都会参与依赖。**

#### 排除传递的依赖项

开发中，我们经常会遇见依赖冲突，对于第三方库导致的依赖冲突，比较好解决，我们只需要使用 `exclude` 解决即可，如下所示：

```groovy
dependencies {
    implementation("androidx.lifecycle:lifecycle-extensions:2.2.0") {
        exclude group: 'androidx.lifecycle', module: 'lifecycle-process'
    }
}
```

#### 统一全局的依赖版本

有时候，某些库会存在好多个版本，虽然 Gradle 会默认选用最高的版本，但是依然不免有时候还是会提示报错，此时我们就可以通过配置全局统一的版本限制：

```groovy
android{
	defaultConfig {
        configurations.all {
            resolutionStrategy {
                force AndroidX.Core
                force AndroidX.Ktx.Core
                force AndroidX.Work_Runtime
            }
        }
     }
}
```

---

### 简化你的BuildConfig配置

开发中，我们常见的都会将一些配置信息，写入到 `BuildConfig` 中，以便我们在开发中使用，这也是最常用的手段之一了。

#### 配置方式1

最简单的方式就是，我们可以在执行 **applicationVariants** task任务时，将我们的 `config` 写入配置中，示例如下：

**app/ build.gradle**

```groovy
android.applicationVariants.all { variant ->
    if ("release" == variant.buildType.getName()) {
        variant.buildConfigField "String", "baseUrl", "\"xxx\""
    } else if ("preReleaseDebug" == variant.buildType.getName()) {
        variant.buildConfigField "String", "baseUrl", "\"xxx\""
    } else {
        variant.buildConfigField "String", "baseUrl", "\"xxx\""
    }
    variant.buildConfigField "String", "buglyAppId", "\"xx\""
    variant.buildConfigField "String", "xiaomiAppId", "\"xx\""
  	...
}
```

> 在写入时，我们也可以通过判断当前的 **build类型** 从而决定到底写入哪些。

##### 优化配置

如果配置很少的话，上述方式写还可以接收，那如果配置参数很多，成百呢？此时就需要我们将其抽离出来了。

所以我们可以新建一个 `build_config.gradle` ,将上述代码复制到其中。

![image-20220115155803121](https://tva1.sinaimg.cn/large/008i3skNly1gyeex30k3qj31nc0gktcs.jpg)

然后在需要的 **模块** 里，依赖一下即可。

```groovy
apply from: "build_config.gradle"
```

这样做的好处就是，可以减少我们 `app-build.gradle` 里的逻辑，通过增加统一的入口，来提高效率和可读性。

---

#### 配置方式2

当然也有另一种方式，相当于我们自己定义两个方法，在 `buildType` 里自行调用，相应的我们将 **config配置** 按照规则写入一个文件中去管理。

示例代码：

**app/ build.gradle**

```groovy
buildTypes {
    // 读取 ./build_extras 下的所有配置
    def configBuildExtras = { com.android.build.gradle.internal.dsl.BuildType type ->
        // This closure reads lines from "build_extras" file and feeds its content to BuildConfig
        // Nothing but a better way of storing magic numbers
        def buildExtras = new FileInputStream(file("./build_extras"))
        buildExtras.eachLine {
            def keyValue = it == null ? null : it.split(" -> ")
            if (keyValue != null && keyValue.length == 2) {
                type.buildConfigField("String", keyValue[0].toUpperCase(), "\"${keyValue[1]}\"")
            }
        }
    }
   release {
    ...
    configBuildExtras(delegate)
    ...
   }
   debug{
    ...
    configBuildExtras(delegate)
    ...
   }
 }
```

**build_extras**

```groovy
...
baseUrl -> xxx
buglyId -> xxx
...
```

上述两种配置方式，我们可以根据需要自行决定，我个人是比较喜欢方式1,毕竟看着更简单，但其实两者的实现方式也是大差不大，具体看个人习惯吧。

### 管理全局插件的依赖

某些时候，我们所有的model,可能都需要集成一个插件，此时我们就可以通过在 `项目build.gradle` 里全局统一管理，而避免到每一个**Gradle** 下去集成：

```groovy
// 管理全局插件的依赖
subprojects { subproject ->
    // 默认应用所有子项目中
    apply plugin: xxx
    // 如果想应用到某个子项目中，可以通过 subproject.name 来判断应用在哪个子项目中
    // subproject.name 是你子项目的名字，示例如下
    // 官方文档地址：https://guides.gradle.org/creating-multi-project-builds/#add_documentation
//    if (subproject.name == "app") {
//        apply plugin: 'com.android.application'
//        apply plugin: 'kotlin-android'
//        apply plugin: 'kotlin-android-extensions'
//    }
}
```

### 动态调整你的组件开关

对于一些组件，在 `debug` 开发时如果依赖，对我们的编译时间可能会有影响，那么此时，如果我们增加相应的开关控制，就会比较好：

```groovy
buildscript {
	ext.enableBooster = flase
	ext.enableBugly = flase

	if (enableBooster)
   	classpath "com.didiglobal.booster:booster-gradle-plugin:$booster_version"
 }
```

如果每次都是静态控制，那么当我们使用 `CI` 来打包时，就会没法操作。所以相应的，我们可以更改一下逻辑：

我们创建一个文件夹，里面放的是相应的忽略文件，如下所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyepjr0pisj30es04cweg.jpg" alt="image-20220115134753904" style="zoom:50%;" />

然后我们更改一下相应的 `buildscript` 逻辑：

```groovy
buildscript {
	ext.enableBooster = !file("ignore/.boosterignore").exists()
	ext.enableBugly = !file("ignore/.buglyignore").exists()

	if (enableBooster)
   	classpath "com.didiglobal.booster:booster-gradle-plugin:$booster_version"
 }
```

> 通过判断相应的插件对应的文件是否存在，来决定插件在CI打包时的启用状态。在CI打包时，我们只需要通过shell删除相应的配置ignore文件或者通过gradle执行相应命令即可。因为本篇是讲gradle的一些操作，所以我们就主要演示一下gradle的命令示例。

#### 定义自己的gradle插件

我们先简单写一个最入门的插件，用来移除相应的文件，来达到开关插件的目的。

```groovy
task checkIgnore {
    println "-------checkIgnore--------开始->"
    removeIgnore("enableBugly", ".buglyignore")
    removeIgnore("enableGms", ".gmsignore")
    removeIgnore("enableByteTrack", ".bytedancetrackerignore")
    removeIgnore("enableSatrack", ".satrackerignore")
    removeIgnore("enableBooster", ".boosterignore")
    removeIgnore("enableHms", ".hmsignore")
    removeIgnore("enablePrivacy", ".privacyignore")
    println "-------checkIgnore--------结束->"
}

def removeIgnore(String name, ignoreName) {
    if (project.hasProperty(name)) {
        delete "../ignore/$ignoreName"
        def sdkName = name.replaceAll("enable", "")
        println "--------已打开$sdkName" + "组件"
    }
}
```

这个插件的作用很简单，就是通过我们 Gradle 命令 **携带的参数** 来移除相应的插件文件。

```groovy
gradlew app:assembleRoyalFinalDebug  -PenableBugly=true
```

![image-20220115143720964](https://tva1.sinaimg.cn/large/008i3skNly1gyecl43dqhj30zg07qq3f.jpg)

具体如图所示：在 **CI-build** 时，我们就可以通过传递相应的值，来动态决定是否启用某插件。

---

#### 优化版

上述方式虽然方便，但是看着依然很麻烦，那么有没有更简单，单纯利用 `Gradle` 即可。其实如果稍微懂一点 Gradle 生命周期，这个问题就能轻松解决。

我们可以在 `settings.gradle` 里监听一下 Gradle 的 **生命周期** ，然后在项目结构加载完成时，也就是 `projectsLoaded` 执行时，去判断一下，如果存在某个参数，那么就打开相应的组件，否则关闭。

示例：

**settings.gradle**

```groovy
gradle.projectsLoaded { proj ->
    println 'projectsLoaded()->项目结构加载完成（初始化阶段结束）'
    def rootProject = proj.gradle.rootProject
    rootProject.ext.enableBugly = rootProject.findProperty("enableBugly") ?: false
    rootProject.ext.enableBooster = rootProject.findProperty("enableBooster") ?: false
    rootProject.ext.enableGms = rootProject.findProperty("enableGms") ?: false
    rootProject.ext.enableBytedance = rootProject.findProperty("enableBytedance") ?: false
    rootProject.ext.enableSadance = rootProject.findProperty("enableSadance") ?: false
    rootProject.ext.enableHms = rootProject.findProperty("enableHms") ?: false
    rootProject.ext.enablePrivacy = rootProject.findProperty("enablePrivacy") ?: false
}
```

执行build命令时携带相应参数即可:

```bash
gradlew assembleDebug -PenablePrivacy=true 
```

### 参考

[Android开发者-配置你的build](https://developer.android.com/studio/build?hl=zh-cn)

> 我是Petterp,一个三流开发，如果本文对你有所帮助，欢迎点赞支持。
