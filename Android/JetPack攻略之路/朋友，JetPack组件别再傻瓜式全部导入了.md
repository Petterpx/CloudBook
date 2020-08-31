# JetPack组件依赖别再傻瓜式全部导入了

你是否在导入JetPack组件时，无从选择，因为有Gradle依赖传递兜底，所以也就放开心去干了。

答应我，朋友，别再这样了，按需所用，导入所需的组件，本篇文章旨在告诉你 JetPack全家桶你该如何选择。



开始之前，加一句，,只要你使用kt，我们就是好胸🐔，ktx天下无敌 👏





JetPack官网链接

https://developer.android.google.cn/jetpack

JetPack & kt

https://developer.android.google.cn/kotlin/ktx



先列出一个最简单的使用方式，如下：

**androidx.activity**

```groovy
    dependencies {
        implementation "androidx.activity:activity-ktx:1.1.0"
    }
```





### Collection KTX

Collection 扩展程序包含在 Android 的节省内存