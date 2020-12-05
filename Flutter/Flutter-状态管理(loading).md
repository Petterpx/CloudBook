# Flutter|关于状态管理，别再被吓着了👏

![image-20201026131503372](https://tva1.sinaimg.cn/large/0081Kckwgy1gk2nz5bklkj30l806t3yj.jpg)

## 导航

本篇是带大家了解并明白Flutter中`状态管理`相关，着眼与实际应用与通俗(说人话)解释，杜绝概念连篇❎。

对应示例代码地址：[Flutter-example-状态管理]()



## 概述

响应式的编程框架中总会有一个永恒的主题--”状态管理“，无论是 React/Vue(前端开发同学肯定了解)，还是 **Flutter**，为了便于共享组件之间的状态，便于在特定时候干特定操作，都会遵循一些特定的约束，而这个约束的过程我们称它为状态管理。



## 没懂？

生而为人，我很抱歉，我是真一下没看懂。

<img src='https://tva1.sinaimg.cn/large/0081Kckwly1gld79xt3prj30c80c8wes.jpg' style="zoom: 50%;float:center;"/>

说实话，我是一个Android开发者，我看到上面的`概述`是一脸懵逼😑，我一时半会没理解，到底什么是状态管理，这个名词太高级了🐔。所以这些概念都先放到一边吧，作为一个非纯前端开发者，我们应该怎么理解呢？

经历了一段时间的 **自我觉悟(自qiqi人)** 后总结：

**注意，这里的widget或者组件可以理解为Android中的View.**

对于一个组件，我们可能有好多种响应状态，比如手势的**按下**时，手势**放开**时，这些不同的**状态**下，我们的`组件`可能会做出一些改变，而作为开发者我们怎么知道它们改变了呢？就是定义一些状态变量去判断，而这些状态变量是由当前 **widget** 自己持有并管理好呢，还是自己只是做一个接收判断，具体还是由父 **Widget** ( **Android** 开发者你可以理解为调用者传递进来)管理呢，具体都是根据实际情况而定。而上述的这个过程我们就可以称之为 `状态管理`。



## Flutter的状态管理

在 **Flutter** 中，我们都知道 **StatefuleWidget** 对应可变组件，那么相应的，它的一些状态应该被谁管理？`Widget本身？父Widget？还是都行呢？`

**最佳解决方式是:**

根据`实际情况`而定。

<img src='https://tva1.sinaimg.cn/large/0081Kckwly1gld76606gej305i05iwed.jpg' style='float:center'/>

这...,额，这个，你可以认为这就是状态管理的`基本宗旨`，在知道宗旨情况下，我们下面来看看 **Flutter** 究竟如何管理。

## 常见的状态管理方式

下面是状态管理最常见的方法：

- **Widget** 管理自己的状态
- 父 **Widget** 管理子 **Widget** 的状态
- 混合管理(子 Widget 和父 **Widget** 都管理状态)

如何决定使用哪种管理方法，下面是官方给出的一些原则以便更准确的做出选择：

- 如果状态是用户数据，如复选框的选中状态，滑块的位置，则该状态最好是由父Widget管理；
- 如果状态是有关界面外观效果的，例如颜色、动画，那么状态最好是由Widget本身来管理；
- 如果某一个状态是不同 **Widget** 共享的则最好是由他们共同的父Widget管理。

往往在Widget内部管理状态封装性会比较好，而在父 Widget 中管理会比较灵活，有些时候，如果不确定到底该怎么去管理，那么推荐的首选是在 `父widget`中管理会显得更为灵活。



## 实践环节

### widget自己管理自己

比如我们有如下一个示例，当我们点击屏幕时，相应的小方块改变颜色和内容, 因为要做到屏幕任意位置点击都可以触发，所以我们选用 **GestureDetector** 手势管理组件。

> 在这个示例中，我们没有太多操作，就是单纯改变文字显示与颜色，所以对于如何显示的这个判断，我们很简单就会定义一个变量，然后在相应的状态下执行相应不同的处理方式即可。
>
> 你可能会有疑问，为什么这么麻烦，的确好像看起来麻烦，我们在Android开发中，通常会直接更新view,相应的，在Flutter中，我们更新一个 **Widget** ，只需要 `setState`，然后我们的 **Widget** 会重新构建，如果以一个 **Android** 开发的思想，我们将这个状态变量提出来，你会发现你和 **Flutter** 好像做的也并无区别，但为什么 **Flutter** 的这种写法反而更为简洁呢.
>
> 请记住，**Flutter**  中的 **Stateful **与 **Stateless Widget** 区分为，有状态信息与无状态信息的两个 **widget** , 对于一个组件(StateFul)而言，它的不同状态决定了它 `最终` 的显示，而在 **Android** 上，这个所谓的状态仅仅只是影响了` view当前` 的一个显示。

具体示例如下：

|                             动画                             |                             代码                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| <img src='https://tva1.sinaimg.cn/large/0081Kckwgy1gka08rcosbg30by0mw7rf.gif' style="float: left; width: 280px; height: 490px;"/> | <img src='https://tva1.sinaimg.cn/large/0081Kckwly1gkcclwempxj30sk0mvdip.jpg' style="float: right; height: 490px;"/> |

---





### 父Widget管理子Widget状态

有些时候，可能某个widget状态 需要在父 **widget** 地方也用到时，这个时候就可以通过下述方式来实现，即间接的通过父 **widget** 管理了我们子 **widget** 的状态。

示例如下：

| 动画                                                         | 代码                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src='https://tva1.sinaimg.cn/large/0081Kckwly1gld5wtkhfag30au0kuhdt.gif' style="float: left; width: 280px; height: 490px;"/> | <img src='https://tva1.sinaimg.cn/large/0081Kckwly1gkcd22wmj2j30pa0m177k.jpg' style="float: center; width=75%; height: 500px;"/> |

---

### 混合管理

有些情况下，我们可能会配合使用，比如下面示例中，手指按下时，我们屏幕中间小方块周围出现一个深红色边框，抬起时，边框消失，点击完成后，方块的颜色改变。

> 我们在父 **Widget** 管理红色边框是否显示，在子Widget控制小方块的颜色改变。

具体示例如下：

|                           动画效果                           |                           父Widget                           |                           子Widget                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![Dec-05-2020 18-31-06](https://tva1.sinaimg.cn/large/0081Kckwly1gld5s07inlg30au0ku4qp.gif) | ![image-20201205183233104](https://tva1.sinaimg.cn/large/0081Kckwly1gld5spudgdj30gl0lyjtt.jpg) | ![image-20201205183441401](https://tva1.sinaimg.cn/large/0081Kckwly1gld5uxxkgjj30lp0nltbt.jpg) |



---





## 参考资料

- [Flutter实战-状态管理](https://book.flutterchina.club/chapter3/state_manage.html)
- [表情包出处](https://fabiaoqing.com/)

