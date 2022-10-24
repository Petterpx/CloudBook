# Kotlin协程|关于 lifecycleScope 的前世今生

提到lifecycleScope,你会想到什么？

这个啊，我不经常用吗，这不是lifecycle-runtime-ktx 里附带的插件吗，这有啥可聊的？

那你能和我聊聊关于这个组件你都知道什么？

> 见名知意，那肯定是与生命周期相关的协程作用域啊。既然是生命周期，原理上肯定和lifecycle有关，也就是onDestory时会自动帮我们cancel掉。从而省去了我们手动去cancel。

看来你也是经常用，那如果让你自己实现一个类似这样的扩展字段呢？



```kotlin
 fun reverseLeftWords(s: String, n: Int): String {
        val size = s.length
        if (size <= n) return s
        val end = s.substring(0, n)
        val start = s.substring(n)
        return start + end
    }
```