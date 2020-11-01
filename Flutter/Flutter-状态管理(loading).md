# Flutter|状态管理而已，必须给你整明白滴👏

## 导航

本篇是带大家了解并明白Flutter中`状态管理`相关，着眼与实际应用与通俗解释，杜绝概念连篇❎。

本篇导图如下：



对应示例代码地址：[Flutter-example-状态管理]()



## 概述

响应式的编程框架中总会有一个永恒的主题--”状态管理“，无论是 React/Vue(前端开发同学肯定了解)，还是Flutter，为了便于共享组件之间的状态，便于在特定时候干特定操作，都会遵循一些特定的约束，而这个约束的过程我们称它为状态管理。



<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk9huvf9c3j30pi0kgwkz.jpg" alt="image-20201101110710606" style="zoom: 25%;" />

## 没听明白？

说实话，我是一个Android开发者，我看到上面的`概述`是不太明白的(自认比较菜)。

所以这些概念都先放到一边吧，作为一个开发者，我们应该怎么理解呢？于是我产生了如下几个问题？

1. 什么是状态管理呢？
2. 为什么要存在状态管理，状态管理的优势？
3. Flutter状态管理？





## 究竟什么的是TM的状态管理？



## 为什么要存在状态管理，优势？



## Flutter的状态管理

在Flutter中，我们都知道 **StatefuleWidget** 对应可变组件，那么相应的，它的一些状态应该被谁管理？`Widget本身？父Widget？还是都行呢？`

**最佳解决方式是:**

根据`实际情况`而定。

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk9ioxhl1ej30pw0ksgsq.jpg" alt="image-20201101113606481" style="zoom:25%;" />

先别急着喷，这句话听起来有点和没说一样，但的确如此，你可以认为这就是状态管理的`基本宗旨`，在知道宗旨情况下，我们下面来看看Flutter究竟如何管理。

### 常见的状态管理方式

下面是状态管理最常见的方法：

- Widget管理自己的状态
- Widget管理子Widget的状态
- 混合管理(子Widget和父Widget都管理状态)

如何决定使用哪种管理方法，下面是官方给出的一些原则以便更准确的做出选择：

- 如果状态是用户数据，如复选框的选中状态，滑块的位置，则该状态最好是由Widget管理；
- 如果状态是有关界面外观效果的，例如颜色、动画，那么状态最好是由Widget本身来管理；
- 如果某一个状态是不同 Widget 共享的则最好是由他们共同的父Widget管理。

往往在Widget内部管理状态封装性会比较好，而在父 Widget 中管理会比较灵活，有些时候，如果不确定到底该怎么去管理，那么推荐的首选是在 `父widget`中管理会显得更为灵活。



## 实践环节

### widget自己管理自己

比如我们有如下一个示例，当我们点击屏幕时，相应的Text改变颜色和内容, 因为要做到屏幕任意位置点击都可以触发，所以我们选用 **GestureDetector** 手势管理组件。

具体示例动画如下：

<img src='https://tva1.sinaimg.cn/large/0081Kckwgy1gka08rcosbg30by0mw7rf.gif' style="float: left; width: 300px; zoom: 67%;"/>



### 父Widget管理子Widget状态

当我们的需求场景是，通过父widget去通知子widget何时更新时，这种状态管理是比较好的方式。比如你有一个悬浮的**floatingActionButton**，通过点击其来通知内部widget改变。





## 使用到的资料

- [Flutter实战-状态管理](https://book.flutterchina.club/chapter3/state_manage.html)
- [表情包出处](https://fabiaoqing.com/)

