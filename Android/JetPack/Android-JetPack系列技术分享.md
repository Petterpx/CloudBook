# Android-Jetpack系列技术分享

![image-20200504183009674](https://tva1.sinaimg.cn/large/007S8ZIlly1gegli0fj0fj30qb0hytc2.jpg)



## Android端架构发展进程

**MVC 与 MVP**

### ![image-20200506090746573](https://tva1.sinaimg.cn/large/007S8ZIlly1geighfdky8j30r50axaan.jpg)

在Android开发中，Android Studio默认生成的就是MVC，View一般是XML，而Control一般是Activity或者Fragment，通常Model 与 Control 之间通过接口关联，当然更多人都是直接对象调用，在Control中获取Model数据，或者发起网络请求，经常会存在model与view关联的情况。这样带来的好处开发时效率高，但是Control动辄近千行代码，后期维护的小伙伴成本很高，不易于扩展，而且存在内存泄漏的风险。

而MVP就是在 MVC的基础上延伸，它相比MVC 多了一个 P层，用于Model与View的桥梁，使得M与V不再直接关联,主要相比MVC减少Activity中的代码。其中View一般为Fragment/Activity/xml,而且只负责UI更新和ui逻辑。它们之间一般通过一个IControl控制接口来分发，每个层都对应着相应的接口。带来的后果就是写起来非常繁琐，而且如果逻辑过多，P层相对来说过重。



**MVVM**

![image-20200505232302623](https://tva1.sinaimg.cn/large/007S8ZIlly1gehzl0vtx3j30km09u0tg.jpg)

MVVM  通过ViewModel来管理View中需要的数据源,使用 DataBing实现与View的双向绑定，从而实现ui变动，数据自动更新，数据更新，ui也自动变化，并且借助于LiveData和ObservableField实现对View生命周期的监测，使得整个框架具有安全性及高效，同时项目中的代码结构更加清晰。避免了MVP和MVC中的一些不足。



## Android JetPack是什么？

Android JetPack是Google 于2018年IO大会上推出的一个   **标准化开发框架**，更是一个完整的体系架构指南，可以使开发者快捷的构建优质的Android应用程序。其中的一些组件可单独采用，并且渐进式的融入你的项目，同时 利用 Kotlin 语言功能，使开发效率更高。它包含了 **Foundation,Architecture,UI,Behavior**

- Foundation 

  基础组件提供了横向功能，例如向后兼容，测试以及Kotlin的支持；

- Architecture

  架构组件用于设计稳健，可测试且容易维护的应用

- Behavior

  帮助集成标准的Android服务，如通知，权限，拍照，下载等

- UI

  提供微件与辅助程序，比如 [Compose](https://developer.android.google.cn/jetpack/compose)，Fragment

相关参考如下：

![image-20200505234740359](https://tva1.sinaimg.cn/large/007S8ZIlly1gei0an6y1gj31980s4464.jpg)

[Android JetPack官网](https://developer.android.google.cn/jetpack)，[官方2分钟简介](https://www.youtube.com/watch?v=LmkKFCfmnhQ&feature=youtu.be)





### 解决了什么问题？

***使 Android 标准化开发成为现实。***

- 加速开发，每个组件都可以单独使用；
- 消除样板代码；
- 构建高质量的应用，减少崩溃和内存泄漏，并向后兼容；

我们先列举几个Android开发中普遍比较头痛的问题：

- Android 配置变更(常见屏幕旋转);
- Fragment 间数据传递及导航;
- Background Jobs
- Paging





我们这次主要从它的架构层开始：

## LifeCycle

Lifecycle的目标只有三个

- 实现生命周期 管理一致性，做到一处修改，处处生效
- 让第三方组件能够 **随时在自己内部拿到生命周期状态**，以便及时叫停 **错过时机的异步业务** 等操作
- 让第三方组件在调试时，能够 **更方便和安全的追踪 事故所在的生命周期源** 

### Lifecycle问世前的混沌世界

假如我们有一个列表视频播放组件 RmxcVideoPlay，为了做到跟随生命周期及时停止或者继续播放，我们分别需要在每个依赖 RmxcVideoPlay的Activity或者Fragment中调用 onResume,onPause等方法。

![image-20200506203330865](https://tva1.sinaimg.cn/large/007S8ZIlly1gej0axr8p7j30mv0fj41e.jpg)

这样的写法有问题吗？没问题，大多数库都是这样干的，但是却会因为后期逻辑复杂度激增导致的容易出错。

**为什么呢？**

因为需要给每个需要RmxcVideoPlay 的Activity 都安排好生命周期调用，一旦Activty或者Fragment变多，就难以保证修改的一致性。再者，如果调试时，我想确保当前的事件源是哪个Activity，而传统的方式只能额外的注入 Activity到RmxcVideoPlay中。

肯定有同学会想 将使用到这个的Activity可以单独写成一个基类，在基类中实现相应的逻辑。但是我们开发中的组件远远不止Video一种，而且Kotlin和Java都不支持多继承，在开发中，且组合优于集成，此外，如果将其写成基类，就会使得其他子类别无选择的继承 它根本用不到的组件。

### Lifecycle问世后的世界

借助于Lifecycle我们子类可以轻松通过一句代码实现  **getLifecycle().addObserver(rmxcVideoPlay)** 生命周期的管理。

### 为什么LifeCycle可以做到呢？

在 Android 中，Activity是一种模板方法模式的实现，也即将窗口管理，多任务管理等开发者不需要关注的细节放在基类中完成，而对子类只暴露生命周期等回调，这样开发者通过继承Activity，就可以拿到简洁的子类模板Activity。同样 LifeCycle也包含着模板方法模式的封装思想,将生命周期的管理封装在基类中，这样子类只需要调用一行方法即可实现。

### Lifecycle运作原理

Lifecycle 基于观察者模式，使得将需要被生命周期调用的第三方组件，作为观察者添加到 Map中，并在生命周期持有者 Activity/Fragment 等发生生命周期节点变化时，通过 事件分发代理者 LifecycleRegister 以遍历的方式通知相应观察者执行相应逻辑。



## LiveData

- 在Lifecycle 的帮助下，实现生命周期管理的一致性，以及可控的作用域。
- **职责十分克制**：仅限于让事件源注入状态，让订阅者观察状态，乃至于还需“单例”的配合 方能正常使用，从而做到让新手老手 都能自然而然的遵循 **从唯一可信源取材，完整正确的状态分发** 这样一种 得以将不可预期的错误降到最少的状态管理理念。
- 即使不使用DataBing,也能使单向依赖成为可能

***LiveData是在架构设计面向标准化 的背景下诞生的***，所以我们主要帮助大家理解，为什么会存在LiveData这样的设计？



### LiveData问世前的混沌世界

比如我们有一个音频播放软件，其中核心的功能是播放控制。

> 1. 当音乐被暂停，播放时，所有涉及播放元素的界面都要切换到对应的状态
> 2. 当切换歌曲时，所有涉及到当前歌曲的界面都要正确展示信息
> 3. 当切换歌曲时，所有涉及专辑列表的界面，都要正确展示每一项的播放状态

|                             首页                             | 专辑列表                                                     | 播放页                                                       |
| :----------------------------------------------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
| ![img](https://images.xiaozhuanlan.com/photo/2019/4f2900dd75695554adbbe42ef01cbab7.gif) | ![img](https://images.xiaozhuanlan.com/photo/2019/3e533b9246b2ba26a8210f672df4db33.gif) | ![img](https://tva1.sinaimg.cn/large/007S8ZIlly1gejq7p6v8fg309u0gb1ky.gif) |

> 截图来自电商软件《FairyTales》，Achieved by **KunMinX**

对于上面的需求，我们很容易会写出一个单例来控制播放，然后对于影响到的页面UI改变，会使用EventBus(**Android事件发布-订阅总线**) 来进行通知，但是实际上一般播放都有好几个模式，比如顺序播放，随机播放等。类似随机播放模式，实际使用的播放列表是一张打乱顺序的列表，是和给用户展示的专辑列表不同。

但上面的这种实现带来的后果是什么呢？

使用EventBus的同学都知道，对于EventBus ，你必须在你每个监听状态的页面，在特定的生命周期节点中 **手动注册和解绑EventBus**, **这使得因疏忽导致的编码不一致 带来的错误概率极大提升**。

并且，在缺乏理念约束的情况下，随意使用EventBus,将难以追踪真正的数据源，比如下面。

借助于EventBus插件，我们得以发现，如果逻辑复杂，情况就是这样，复杂度 n²

![image-20200507120557030](https://tva1.sinaimg.cn/large/007S8ZIlly1gejr94mpktj30js0b4n02.jpg)

![image-20200507140306673](https://tva1.sinaimg.cn/large/007S8ZIlly1gejun15zfej30pu0ac3z6.jpg)



如果采用上面的方式，我们寻找问题时难免欲仙欲死，并且，通过EventBus从页面发送通知，无法确保 **消息的准确性和可靠性** ，也许当前页面信息已经是过时的了，那么接下来的所有收到通知的页面，拿到的状态也都是过时的，这非常影响用户体验。 



### LiveData问世后的世界

在这种状态下，Google推出了LiveData, LiveData诞生的背景就是 ：**Google希望确立一种标准化，规范化的开发模式，就算是初学者，也能自然而然的遵循这样一种模式.**于是，LiveData 就是在这样的背景下诞生。

为了达到这个目的，LiveData 被设计为仅限于负责 **状态在订阅者生命周期内的分发(借助于Lifecycle)**,因此LiveData的方法很简单，只有setValue/postValue,以及observer，除此之外，再也看不到别的方法。

**正因为如此纯粹，甚至无法独当一面，以至于LiveData的使用总是伴随着ViewModel 这样的 单例**，**来完成跨层 (从model到view)的状态分发**。

> 只要你这样做了，便不知不觉达成了两个目标：

1. 一个是单向依赖： UI-> ViewModel -> Model；
2. 另一个是 **无论是哪个页面发起的对状态改变的请求，最终所有页面的状态改变都来自一个确定，唯一的事件源；**

![image-20200507140328865](https://tva1.sinaimg.cn/large/007S8ZIlly1gejunevhbkj30n50c2aay.jpg)

如此一来，我们代码的复杂度及调试时的难度都大大降低，再者，得益与 Lifecycle，**订阅者只能接收来自 onResume-onPause节点内的状态分发，从而有效避免了因为当前页面被Destory而导致的null指针异常，同时又不影响其他处于可视状态下的订阅者的状态分发。**而且会随着页面的销毁，订阅者也会自动解绑，无需开发者手动编写，避免了因为疏忽而导致的不必要错误。

**如下代码所示**

<img src="/Users/petterp/Downloads/carbon%20(4).png" alt="carbon (https://tva1.sinaimg.cn/large/007S8ZIlly1gejvgxurljj30tq136n21.jpg)" style="zoom:50%;" />







## ViewModel

> ViewModel的作用如下用来保存应用UI数据的类，并会在配置更改后继续存在；

提到ViewModel,大家可能通过最前面的架构图，很容易觉得它好像是MVP中Presenter的代替产物，这就使的ViewModel的价值没有被真正的重视和利用，而且相对来说，ViewModel好像也没什么要讲的。

在JetPack中，讨论LiveData总是离不开ViewModel,而ViewModel作为中间层确保着UI数据的完整性。

在Android开发中，特别是MVC中，很常见的会将一些逻辑和UI数据都放在Activity/Fragment中，这种做法违背了单一原则，导致Activity/Fragment变得臃肿。**而ViewModel可以有效做到职责划分，我们可以将所有的UI数据都放到ViewModel中，需要注意的是Activity仅负责了解如何在屏幕上显示该数据及接受用户互动，但它不处理具体的互动信息，要处理这些逻辑，建议创建一个Presenter类。**

![image-20200507151329536](https://tva1.sinaimg.cn/large/007S8ZIlly1gejwo9gjp1j30dr0by74t.jpg)

于是，结合上面的LiveData，我们就可以写成下面这样的代码。

|                             xml                              |                           Activity                           |                          ViewModel                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20200507161936017](https://tva1.sinaimg.cn/large/007S8ZIlly1gejyl1sh2sj30gl0fitaw.jpg) | ![image-20200507162023036](https://tva1.sinaimg.cn/large/007S8ZIlly1gejyluunw3j30ly0dj0um.jpg) | ![image-20200507162103479](https://tva1.sinaimg.cn/large/007S8ZIlly1gejymk53g2j30ej08hgm7.jpg) |





## JetPack在公司项目中的实践

首页滑动时：

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gejzbdlvkhg30u01o04r6.gif" alt="审判gif" style="zoom: 25%;" />

在首页下拉滑动时，以往的写法都是自定义个刷新接口，让Activity实现此接口，然后在子Fragment中强转拿到接口对象，整个过程因为布局是Activity嵌套Tab-Fragment+子Fragment，所以在Activity中，还需要调用TaB-Fragment的方法，实现起来略显麻烦，当使用LiveData+ViewModel时，实现起来就非常简单，**而且每个页面在销毁时相对应的观察者也会自动解绑，从而避免了内存泄漏的问题。**

网评IM音视频通话成员邀请时，群聊解散命令通知时等；

人民融云架构雏形图如下：

| 架构                                                         | Base层-MVVM                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20201122100610908](https://tva1.sinaimg.cn/large/0081Kckwly1gkxq3u0o3jj30pi0nijte.jpg) | ![image-20200507180443748](https://tva1.sinaimg.cn/large/007S8ZIlly1gek1mft5coj30j90grwft.jpg) |

