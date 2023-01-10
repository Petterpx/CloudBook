# 关于折叠屏适配，你应该了解的一切




最近接到了某厂商关于折叠屏的适配通知，并指出了我们app中存在的不兼容页面，于是乎，就此记录一下过程。

作为一个技术同学，我所了解的以下内容，可能会对你有所帮助。

## 引言

折叠屏适配，是一个比较复杂的事情，从2019年开始，关于折叠屏的适配就沸沸扬扬，但是雷声大雨点小，真正做到适配的并不大多。

一方面是因为用户量并不大，另一方面就是需要重新设计交互。所以目前的折叠屏适配大多数都只是 **兼容**，而非 **适配**。

所以如果某一天，产品过来告诉你，适配一下折叠屏，相信我，这并非一个简单事情。



## 非技术角度看待适配





## 技术指南

从Android10开始(Api29)，google提供了为可折叠设备和不同折叠模式增加支持的api，以便为可折叠设备和不同折叠模式增加更多的支持。

**api方面**

当应用在Android10上运行时，**onResume()** 和 **onPause()** 方法的工作原理并不相同。当多个应用同时处于多窗口模式或多显示屏模式时，可见堆栈中所有可设置为焦点的顶层 Activity 都处于“已恢复”状态，但实际上焦点仅位于其中一个 Activity 上，即“在最顶层处于已恢复状态”的 Activity。在 Android 10 之前的版本中运行时，一次只能恢复系统中的一个 Activity，而所有其他可见 Activity 都处于已暂停状态。

请不要将“焦点位于”的 Activity 与“在最顶层处于已恢复状态”的 Activity 混淆。系统会根据 Z-Order 来为 Activity 分配优先级，以便为用户最后进行互动的 Activity 提供更高的优先级。Activity 可能在顶层处于已恢复状态，但焦点却并不位于其上（例如，如果通知栏展开）。

在 Android 10（API 级别 29）及更高版本中，您可以订阅 [`onTopResumedActivityChanged()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onTopResumedActivityChanged(boolean)) 回调，以便在 Activity 获取或失去在最顶层处于已恢复状态的位置后收到通知。这相当于 Android 10 之前版本中的已恢复状态；如果您的应用使用的专用或单一资源可能需要与其他应用共享，这可以作为有用的提示。

[`resizeableActivity`](https://developer.android.com/guide/topics/ui/multi-window?hl=zh-cn#resizeableActivity) 清单属性的行为也发生了变化。如果某个应用在 Android 10（API 级别 29）或更高版本中设置 `resizeableActivity=false`，则当可用屏幕尺寸发生变化或者该应用从一个屏幕移到另一屏幕时，它可能处于兼容模式下。



应用可使用 Android 10 中引入的 [`android:minAspectRatio`](https://developer.android.com/reference/android/R.attr?hl=zh-cn#minAspectRatio) 属性来指示您的应用支持的[屏幕宽高比](https://developer.android.com/preview/features/foldables?hl=zh-cn#new_screen_ratios)。





### 应用连续性

在可折叠设备上运行时，应用可以自动从一个屏幕转换到另一个屏幕，为了提供更好的用户体验，务必确保当前任务能在转换后继续无缝执行。应用应在同一个位置以相同状态恢复。





在一定程度上，对于某些页面，这个方法或许会很有用，比如某个直播详情页：

非折叠模式只展示直播，折叠模式展示详情购买页面：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsfk6o16cfg30hs0hsnpd.gif" alt="图片" style="zoom:50%;" />

> 图片来源：[淘宝设计-不要惊讶，折叠屏就应该这样设计！](https://mp.weixin.qq.com/s/MKC3z-yhqv_o6LtVj-DKuQ)





### 开发过程中如何调试

#### 使用模拟器调试

Android studio3.5 开始，支持创建折叠屏的虚拟设备，如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsfj2piot4j317u0u0jxb.jpg" alt="image-20210713190524417" style="zoom:33%;" />

#### 通过adb调试

> 需要注意的是,需要在开发者选项中开启，usb调试(安全设置)

模拟 1800*1700 分辨率

```shell
adb shell wm size 1800x1700
adb shell wm size 2480x2200
```

分辨率恢复

```shell
adb shell wm size reset
```



看完了，他的第一个版本那里估计写错了(model可变了)，seales class 这个思路没问题，通过高内聚的方式解决。但总结的 息屏后的这里，不敢苟同，这是使用方的问题，他因为考虑到了多个消息的丢失，于是希望将状态记录下来，再还原，按照这个设计理念，在繁琐的业务下，势必要还原一遍所有状态，或者会放弃某些状态。本质上这种方式属于过度设计，因为对于view而言，其实在他的背景下(息屏-亮屏后)，只关心最终显示状态。

## 技术方案

#### 允许 resizeable(可变尺寸)

即允许可变尺寸，Android7.0 以上，默认支持可变屏幕，即app会在任何分辨率情况下，都按照全屏显示，当然这可能会出现一些UI变形，这就需要开发者自己处理好。



#### 使用备用布局

比如layout中，你可以创建如下文件夹

![image-20210713191844161](https://tva1.sinaimg.cn/large/008i3skNly1gsfjgkksvrj30m6064jrs.jpg)

具体对应关系如下：

- small、normal、large、xlarge：屏幕大小

- long、notlong：屏幕类型-长屏幕或非长屏幕

- port、land：纵向 或 横向

- car、desk：扩展坞类型

- night、notnight：深色 或 非深色

- ldpi、mdpi、hdpi、xhdpi、nodpi、tvdpi：屏幕像素密度

这些之间的配置，使用 **-** 连接。





虽然适配才是最终方案，但碍于一些原因，很可能没办法及时适配，那么google提供了以下方法以供选择。

#### 声明受限屏幕

- 如果app -targetSdkVersion>=24 ，则默认会支持 resizeable
- 如果app -targetSdkVersion<24,需要在manifest中显式声明，才可以支持 resizeable

如果你不想适配折叠屏，那么只能使用下述方式，即硬编码的方式，始终保证屏幕的比例展示正常：

在mainfest-activity中增加以下标签：

> 增加到application中，则为全局设置

```xml
android:resizeableActivity="false"
android:maxAspectRatio="2.4"
android:minAspectRatio="1.1"
```

**resizeableActivity** 禁止分屏

**maxAspectRatio** 最大屏幕尺寸(Android10及以上)

**minAspectRatio** 最小屏幕尺寸(Android10及以上)

Android官方，目前建议屏幕尺寸支持范围为1.1到2.4之间。

一旦声明了受限屏幕的的比例，当app运行在一个屏幕比例超出声明范围时，app将在屏幕上显示黑边等信息。如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsfiyjoilkj31cw0ta777.jpg" alt="image-20210713190122208" style="zoom:50%;" />