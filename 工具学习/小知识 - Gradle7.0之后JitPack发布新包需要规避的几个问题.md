# 小知识 - Gradle7.0之后JitPack发布新包需要注意的几个问题



最近在使用 `JitPack` 发布组件时候，遇到了这几个问题,着实找了好一会才解决，分享一下。🙃

### 问题1 - 调整jdk版本为11

* What went wrong:
An exception occurred applying plugin request [id: 'com.android.application']
> Failed to apply plugin 'com.android.internal.application'.
> Android Gradle plugin requires Java 11 to run. You are currently using Java 1.8.
> You can try some of the following options:
>      - changing the IDE settings.
>           - changing the JAVA_HOME environment variable.
>           - changing `org.gradle.java.home` in `gradle.properties`.

这个问题，我们如果是在本地Android Studio，很简单，直接改 `gradle` ,但是 `jit` 上呢😶，解决办法如下：

创建一个 `jitpack.yml` 文件,具体如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvdia9pv0mj61400oemzb02.jpg" alt="image-20211013110754893" style="zoom:67%;" />

`jitpack.yml`

```xml
jdk:
  - openjdk11
```



### 问题2 - 无法创建 AndroidMavenPlugin 插件

* What went wrong:
A problem occurred evaluating script.
> Failed to apply plugin 'com.github.dcendents.android-maven'.
> Could not create plugin of type 'AndroidMavenPlugin'.
>     > Could not generate a decorated class for type AndroidMavenPlugin.
>        > org/gradle/api/publication/maven/internal/MavenPomMetaInfoProvider

解决办法：

打开相应的组件的 **build.gradle** (如果你的组件都写在app里了，那就只写到app里即可)，增加以下内容：

```groovy
plugins {
   	...
    id 'maven-publish'
}

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'com.petterp'
                artifactId = 'statex'
                version = '1.0-dev02'
            }
        }
    }
}
```

- `groupId` 是你的包名除 labrary 之前的全称
- `artifactId` 对应 **labrary id** ,即组件ID
- `version` 版本号

**示例**

比如上述全称是 `com.petterp.statex` , `statex` 是我的组件 **library**，故就是上述这样的写法。

当然对于 `version` 这些统一管理 你也可以配置到专门的 `gradle` 文件中，这里直接引用即可，这里就不多赘述了。