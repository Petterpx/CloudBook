# Gradle系列之 init.gradle

## 初始化脚本

初始化脚本，又名 init scripts，大致和我们其他见到的gradle脚本类似，但这个脚本会在项目构建开始之前运行，官方推荐的常用场景如下：

- 用于建立公司内部的配置，定义公司内部的仓库地址
- 根据当前的环境配置一些全局属性，比如配置一些key的地址等配置
- 用来提供构建者所需要的用户信息，比如仓库或数据库的用户名及密码等等
- 用来定义开发者机器的环境，比如jdk的安装位置，android sdk的安装位置等等
- 用来注册一些监听器，比如监听gradle事件的发生，做一些额外的操作
- 对构建日志进行重定向等

但需要注意的是，其无法访问buildSrc 项目中的类：

> 主要原因在于，buildSrc 在gradle里是 **复合构建** 的一部分，它并不属于当前project,而是gradle在编译中只要发现该目录，gradle就会自动编译并测试其中的代码，然后将其放入构建脚本的类路径。对于多项目而言，只能存在一个 buildSrc 目录，并且该目录必须位于项目的根目录中。 [具体🔗](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)
>
> 所以因为其在构建上并不属于当前项目，而init.gradle只是基于当前项目执行，所以在执行时无法访问 buildSrc 项目里的类。
>
> **什么是复合构建？**
>
> > 复合构建是包含其他构建的一个构建。大多数情况下，复合构建类似于gradle多项目构建，不同之处在于它包括了完整的build记录，而不是只有当前projects对应的build。[具体🔗](https://docs.gradle.org/current/userguide/composite_builds.html#composite_build_intro)

## 如何使用 init 脚本

1. 通过 -I 或者 –init-script 参数在构建开始时指定路径，如

   - gradle --init-script init.gradle clean
   - gradle -I init.gradle assembleDebug

   需要注意的是，上述的命令可以出现多次，比如 下述示例：

   ```groovy
   gradlew -I init1.gradle -I init2.gradle assembleDebug
   ```

   每次添加另一个初始化脚本，如果命令行上指定的任何文件不存在，则构建将失败。

2. 在 USER_HOME/.gradle/ 下放一个 init.gradle 文件

3. 在 USER_HOME/.gradle/init.d/目录下 存放多个 .gradle 结尾的文件

4. 在 GRADLE_HOME/init.d/目录下 存放多个 .gradle 结尾的文件

Gradle会依次对上面的目录进行检测，如果找到多个，则按照脚本名称的字母顺序执行。比如你可以在命令行里指定一个init脚本，而同时也可以执行用户自己目录里的脚本，这两个脚本将在gradle执行时顺序执行。

### 获得自己的路径

一般而言，USER_HOME 指的是用户目录下的 .gradle 目录，而 GRADLE_HOME 目录指的gradle的可执行目录。

如下所示：

- `gradleHomeDir:` /Users/petterp/.gradle/wrapper/dists/gradle-7.2-bin/2dnblmf4td7x66yl1d74lt32g/gradle-7.2
- `gradleUserHomeDir:` /Users/petterp/.gradle

**打印gradle的配置**

可以通过以下代码打印自己的配置信息

```groovy
// gradle的可执行目录
gradle.println "gradleHomeDir:${gradle.gradleHomeDir}"
// gradle的用户目录,缓存一些下载好的资源，编译好的构建脚本等
gradle.println "gradleUserHomeDir:${gradle.gradleUserHomeDir}"
//gradle的版本号
gradle.println "gradleVersion:${gradle.gradleVersion}"
//gralde当前构建的启动参数
gradle.println "startParameter:${gradle.startParameter}"
```

## 常见场景

### 1.为项目添加默认仓库

init.gradle

```groovy
allprojects {
    repositories {
        mavenLocal()
    }
}
```

项目build.gradle 下增加以下代码用于测试：

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '7.2.0-beta04' apply false
    id 'com.android.library' version '7.2.0-beta04' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
}

repositories {
    mavenCentral()
}

//增加以下代码用于测试
tasks.register("showRepos") {
    it.doLast {
        println "打印当前存储库"
        println repositories.collect { it.name }
    }
}
```

执行下述命令测试效果：

```groovy
// 已经将init.gradle 放到 /Users/xxx/.gradle/下
gradlew -q showRepos

// 通过命令行执行
gradlew --init-script  init.gradle  -q showRepos
```

**结果：**

```
打印当前存储库
[MavenLocal, MavenRepo]
```

### 2.使用三方库

```groovy
initscript {
    //定义构建脚本所需要的依赖地址
    repositories {
        mavenCentral()
    }
    //加入依赖
    dependencies {
        classpath 'org.apache.commons:commons-math:2.0'
    }
}
//使用函数
println org.apache.commons.math.fraction.Fraction.ONE_FIFTH.multiply(2)
```

```
结果：
2 / 5
```

### 3.初始化脚本插件

有时候我们想限制构建时仅使用指定的存储库,就可以编写如下的示例逻辑：

```groovy
apply plugin: EnterpriseRepositoryPlugin

class EnterpriseRepositoryPlugin implements Plugin<Gradle> {

    private static String ENTERPRISE_REPOSITORY_URL = "https://repo.gradle.org/gradle/repo"

    void apply(Gradle gradle) {
        // ONLY USE ENTERPRISE REPO FOR DEPENDENCIES
        //限制构建时仅使用指定的存储库
        gradle.allprojects { project ->
            project.repositories {
                // Remove all repositories not pointing to the enterprise repository url
                all { ArtifactRepository repo ->
                    if (!(repo instanceof MavenArtifactRepository) ||
                            repo.url.toString() != ENTERPRISE_REPOSITORY_URL) {
                        project.logger.lifecycle "Repository ${repo.url} removed. Only $ENTERPRISE_REPOSITORY_URL is allowed"
                        remove repo
                    }
                }

                // add the enterprise repository
                maven {
                    name "STANDARD_ENTERPRISE_REPO"
                    url ENTERPRISE_REPOSITORY_URL
                }
            }
        }
    }
}
```

### 4.更改Gradle默认日志

默认的 assembleDebug 执行时控制台会默认输出一系列log,如果我们想

```groovy
useLogger(new CustomEventLogger())

class CustomEventLogger extends BuildAdapter implements TaskExecutionListener {

    void beforeExecute(Task task) {
        println "[$task.name]"
    }

    void afterExecute(Task task, TaskState state) {
        println()
    }

    void buildFinished(BuildResult result) {
        println 'build completed'
        if (result.failure != null) {
            result.failure.printStackTrace()
        }
    }
}
```

#### 注册gradle构建监听

```groovy
gradle.addBuildListener(new BuildListener() {
    @Override
    void beforeSettings(Settings settings) {
        gradle.println("settings.gradle加载前调用")
    }

    @Override
    void settingsEvaluated(Settings settings) {
        gradle.println("settings.gradle加载与评估完成")
    }

    @Override
    void projectsLoaded(Gradle gradle) {
        gradle.println("项目加载完成")
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
        gradle.println("项目评估配置结束")
    }

    @Override
    void buildFinished(BuildResult buildResult) {
        gradle.println("构建完成")
    }
})
```