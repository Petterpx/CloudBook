# Flutter入门笔记-初识



## What is Flutter?

Flutter 是 Google 推出并开源的移动应用开发框架(简称UI框架)，主打跨平台，高性能。Flutter 使用 Dart语言作为开发语言，一套代码可以同时运行在  ios 和 Android 平台，现在也支持 mac,windows. 

## Flutter相比其他框架的优势？

### 跨平台自研引擎

Flutter 使用自己的渲染引擎来绘制 widget ，而非WebView (**H5**) 或者Android系统原生的控件(**RN->自定义View**)，这样不仅可以保证在Android 和 iOS 上UI的一致性，同时也避免对原生控件依赖而带来的限制和维护成本。

### 高性能

- Dart -> 开发语言
- 自渲染引擎

Flutter 高性能主要靠上面两点来保证，首先，Flutter 采用Dart语言开发，Dart支持AOT模式，其次，Flutter使用自研引擎来绘制UI，布局的数据直接由Dart 控制。所以布局过程中无需与Native通信，特别在滑动场景下，Flutter相比js或者 其他需要与Native之间不停同步布局信息的框架相比，会带来比较客观的性能提升。



## 从Android开发者的眼中去看Flutter

Flutter 自17年发布，到18年发布正式1.0版本，到现在1.22版本，已经非常稳定了。从生态上看，使用Flutter的用户也越来越多，相关的资料也开始逐渐丰富，国内 咸鱼技术团队，今日头条技术团队等都早已有了自己的Flutter技术栈，相关的问题早已在Stackoverflow有太多解释。 对于日渐平稳的原生开发来说，学习Flutter不尝是一件有意思的事。

### Flutter上手是不是很难？

作为一个Android开发者，在2020的今天，Kotlin早已是我们的第一开发语言，语言层我们都是从Java->Kotlin，当你再去学习Dart时，会发现非常简单，Dart 就像一个融合了多种语法的语言，所以我们不难可以猜想，Google 可能是想把Dart打造成一门集百家之所长的语言。

[Flutter for Android开发者](https://flutterchina.club/flutter-for-android/)

我个人建议，如果你有丰富的开发经验，完全可以大体看一遍上述文档，然后就可以开始学习Flutter了，在实际运用学习Dart。





## Flutter框架图

![1-1](https://tva1.sinaimg.cn/large/007S8ZIlly1gjmvgdzlekj30mn0ch0ti.jpg)



### Flutter FrameWork

- 底下两层(Foundation 和 Animation,Painting，Gestures) 对应的是Flutter中的 dart:ui包，它是Flutter引擎暴露的底层UI库，提供动画，手势及绘制能力。
- Redering层，这一层是一个抽象的布局层，它依赖于dart UI层，Redering层会构建一个UI树，当UI树有变化时，会计算出有变化的部门，然后更新UI树，最终将UI树绘制在屏幕上，这个过程类似于React中的虚拟DOM。Rendering层可以说是Flutter UI框架最核心的部分，它除了确定每个UI的位置，大小之外还要进行坐标变换，绘制(即调用底层dart:ui).
- Widget 层是Flutter提供的一套基础组件库，在基础组件库之上，Flutter还提供了 Material 和 Cupertino 两种视觉风格的组件库，而我们 Flutter 开发的大多数场景，只是和这两层打交道。

### Flutter Engine

一个纯的C++实现层，其中包括了 Skia 引擎，Dart运行时，文字排版引擎等。在代码调用 **dart:ui** 库时，调用最终会做到 Engine 层，然后实现真正的绘制逻辑。