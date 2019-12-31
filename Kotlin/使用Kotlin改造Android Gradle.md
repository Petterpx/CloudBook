# 使用Kotlin改造Android Gradle

![image-20191231143912761](https://tva1.sinaimg.cn/large/006tNbRwly1gafwd6621jj30vg0hmk0n.jpg)

[目前Gradle官方已经将这个计划加入进展中，但不建议开发使用，尝鲜即可。](https://github.com/gradle/kotlin-dsl-samples)

### 为什么要改造？

```
Gradle采用groovy采用开发语言，是一种动态的dsl语言，缺点就是写脚本时如果出现问题，我们无法实时的得知，只能通过print进行得知。而且无法跳转，并且不支持自动补全。
```

我们先建一个普通的Android项目，然后改造如下：

**改造很简单，在你的gradle后面加上kts即可。**

![image-20191231142859095](https://tva1.sinaimg.cn/large/006tNbRwly1gafw2i7p2mj30ct0g4dgl.jpg)



#### app的gradle

```kotlin
plugins {
    id("com.android.application")
    id("kotlin-android")
    id("kotlin-android-extensions")

}


android {
    compileSdkVersion(29)
    buildToolsVersion("29.0.2")
    defaultConfig {
        applicationId = "com.android.daggertest"
        minSdkVersion(15)
        targetSdkVersion(29)
        versionCode = 1
        versionName = "1.0"
    }
    buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro")
        }

    }

    dependencies {
        "implementation"(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
        "implementation"("org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.61")
        "implementation"("androidx.core:core-ktx:1.1.0")
        "implementation"("androidx.constraintlayout:constraintlayout:1.1.3")
    }
}
```



#### 项目的Gradle配置

```kotlin
buildscript {
    val kotlinVersion="1.3.61"
    repositories {
        google()
        jcenter()

    }
    dependencies {
        classpath("com.android.tools.build:gradle:3.5.1")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()

    }
}
```

#### model配置

```kotlin
include("app")
```



这样改造后就ok了，不过需要注意的是，目前Android  Studio对Kotlin-Gradle支持的并不是很好，所以如果你新建一个model,就会再次新创建gradle配置文件，这样你就得再次手动配置，很麻烦，所以目前Kotlin-Gradle尝鲜即可。







#### [参考链接：](https://juejin.im/post/5a6ab7a6518825733707dd55)