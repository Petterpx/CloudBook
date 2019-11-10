# Activity加载流程

![image-20191018224723553](https://tva1.sinaimg.cn/large/006y8mN6ly1g82qmgql1xj30cg0ey3zs.jpg)



开始之前，我们首先需要明白一些基础概念，Activity与Window的关系是什么？扩展开来就是 Activity与Window和 DecorView 之间的关系。从而再来分析 Activity 的加载流程。

观察上图，我们很容易看出，他们之间大体的关系，下面我用一些简单话语的梳理一下，帮助大家形成一个基础的概念框架。

在Android中，Window表示一个窗口的概念，在日常开发中，我们直接使用Window的机会并不多，但我们常使用的Toast,Dialog等都与它有关。

window是一个抽象类，PhoneWIndow是它的一个具体实现类，每一个Activity都持有一个 window对象，也就是每一个Activity都持有一个 PhoneWindow对象。继续向下延伸，每个PhoneWindow 内部都持有一个 DecorView，我们Activity中大多数View操作都是通过 DecorView来完成。







首先，本次分析源于API29  Android Studio3.5



我们从 setContView方法开始：

我们可以发现，内部调用了getWindow.setContentView(),然后传入了我们的布局。

```
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```

继续进setConteView方法

