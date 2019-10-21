# 基于LiveData+ViewModel+Retrofit的网络请求库。

这篇不同于大多数博文，我会尽量用一个读者的思想去写，重要的不是代码，而是我们对整个过程有自己的一个认知。

**关于一些基本疑问点：**

> 什么是liveData与ViewModel,google为什么推荐使用？

> 我为什么要使用 liveData+ViewModel 来封装网络请求库，RxJava不行吗？

> 相比起普通的封装Retrofit ，使用 LiveData+ViewModel 有什么好处？

> 你如何保证你的封装是否合理，具有一个工具基本的适用性？

**深入了解？**

> [LiveData中的 onActive()和 onInactive() 方法是什么用](https://www.jianshu.com/p/958b433332f5)

> 我现在是MVP的框架，LiveData+ViewModel 应该如何与我现有的框架融合？

> 能否画出你的 调用顺序，即一个完整的请求过程？它与普通的封装请求过程不同点在哪？

> addCallAdapterFactory 和 addConverterFactory的区别在哪里？

在我开始前，这些是我自己提出的一些问题，首先我需要自己明白，这个工具是干什么的？它和别的工具相比，优势在哪里，它的整体逻辑，解决的问题，我需要明白，然后才能开始着手封装，我希望大家在看的时候，也能抱有一些疑问，为什么要这样？这样有什么好处？

