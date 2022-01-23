# 山川湖海 - Android无障碍功能优化实践



> 是谁来自山川湖海，却囿于昼夜、厨房与爱
>
> 《万能青年旅店乐队》

Hi，很高兴见到你！👋🏻

本文主要分享Android无障碍功能的一些优化经验，希望看完本篇，可以帮助到你，以及哪些特殊的用户。

## 前言

最近我们团队收到了一些用户的反馈：

![image-20220123103209858](https://tva1.sinaimg.cn/large/008i3skNly1gyneggvprlj31hi0dadjv.jpg)

有用户反馈，我们的部分功能按钮在无障碍下无法正常识别。其实这已经不是我第一次看到反馈了，第一次是16年ios端收到了用户的反馈，进行过一次优化。说来惭愧，反而是我们 Android 这边也一直没有专门进行过适配。

## 什么是无障碍功能？

对于一些视障人群或者听障人群而言，普通的App对它们来说使用起来可能困难重重。在 `Android` 上，对于这些用户用户而言，主要通过系统附带的屏幕阅读器 `TalkBack` 来进行控制设备。

所以无障碍功能是应用开发中的重要组成部分，通过集成无障碍功能和服务，可以提高应用的易用性尤其是对于残障用户而言。

## 背景与现状

> 我国现在 `1691万` 视障人士，`2780万` 听障人士，`2977万` 肢体残障人士，数据来源于 **2021年第7次** 人口普查，中国互联网络信息中心官网。

在国内，专门去处理的并不太多，一是因为这件事情很多开发者并不知晓(我们下面会提到为什么)，再者相对而言的收益可能并不高及一般也没有用户反馈，这件事情就一直没有太被重视，对于 **无障碍功能** ，可能更多是部分工具App中常见，比如某些抢红包等插件。

而 **无障碍功能** 的适配在国外却是相对比较常见的一个事，甚至于某些国家如果不做适配可能会无法上架；

纵观业内，腾讯系产品在这方面做的比较好，当然这与他们内部标准的开发规则及庞大的 **用户群体** 也有关系。

**那为什么大多数开发者不知晓呢？**

其实主要源于 `Android原生UI` 对于 **无障碍** 并不是 `强制性` 的，所以大多数国内开发者在初入 Android 时甚至都没法关注到 **无障碍功能** 这个事，所以一直导致在国内这个事情似乎并不是那么重要，如下所示：

我们一般都会将布局写在 `xml` 中，默认编译器也会提示我们，但因为其不是强制性，所以如果你不点提示(option+回车)，似乎根本不会涉及到[`contenDescription`] ,如下所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gykgdrv1zmj31yo0jedlh.jpg" alt="image-20220120212204330" style="zoom:50%;" />

这里并不是官方不作为，相反，其实 `Android` 团队也一直在改善这方面体验，编译器已经尽可能在 **提示** 我们做一些优化工作。 但更多的是因为这和 `Android原生UI` 在 **无障碍** 上根深蒂固的 **开发模式** 有关系，即 `非[显式]` 。

## 技术支持方面

一直以来，Android团队 都建议我们去适配无障碍功能，甚至于官方有专门的网站及文档用于描述，在每年的IO大会上，也都有提及：

[Android原生UI-构建无障碍功能更出色的应用](https://developer.android.com/guide/topics/ui/accessibility/)

[Compose - 对于无障碍的支持](https://developer.android.com/jetpack/compose/accessibility)

[GoogleIO-2021-无障碍分场](https://io.google/2021/program/products/accessibility/?lng=zh-CN)

**那无障碍适配是不是很困难啊？**

并不是，正是由于官方的支持，我们开发者要做的基本上很少。

对于 `Android原生UI` 而言，如果应用主要使用的是 [系统组件]，那么在无障碍下，体验一般不会太差，比如常见的 `Text` , `Button` 。在无障碍下都会读取相应的显示文本信息作为描述。

**如果我使用Compose进行UI开发呢？**

似乎 Android团队 也发现了这个过去 **非强制性** 的问题，可能也得益于声明式开发的便捷。

与 `原生UI` 相比，`Compose` 在无障碍上的要求就 **[严格]** 了不少，如果你使用的是 `非Text` 组件，那么必须传递相应的 `contentDescription` ,当然这个值也可以传递为 null ,但是你必须 **显式** 的去声明，如下所示：

![image-20220121091708582](https://tva1.sinaimg.cn/large/008i3skNly1gyl11swivlj315y088t9v.jpg)

所以技术层面完全不是问题，我们开发者要做的就是在开发中注意一些按钮或者给图片增加一些描述即可，这样就能很低成本的做到适配无障碍功能。

## 无障碍模式开发准则

- 遵循 `Material` 中的无障碍开发规则
- 为 `非Text` 类的 `View` 增加合适的 `contentDescription`
- 对于一些 `装饰性的UI元素` 去掉标签及焦点
- `EditText` 通过 `hint` 设置标签
- 比较复杂的页面采用 `分组聚集` 的方式
- 对 **自定义** 的 View 进行无障碍适配

## 适配技巧

通过下面的技巧，便于你快速掌握适配方式，落地到开发中。

> ps:这里会持续更新，在业务中有更多实践后也会同步到这里。

### 为你的View增加描述

对于继承自 `TextView` 类的组件，Android框架本身可以读出文本的信息，所以一般情况下，我们无需再次手动适配，我们主要需要适配的是哪些`Image` 或者 **无法描述** 的一类组件。

如下所示，增加相应的描述即可。

```xml
android:contentDescription="xxx"
```

```kotlin
view.contentDescription="xxx"
```

---

### 使用ImageBtn替代Image

对于按钮(**即可以交互的一些行为**)，Android官方建议使用 `ImageButton` 替代普通的 `ImageView` 。

相信不少同学在定义自己Bar时，肯定使用的 `Image` 作为返回按钮，这也是很常见的，但为什么官方建议大家使用 `ImageButton` 呢？

主要是因为在适配无障碍模式时，无障碍服务在读取到 `Image` 时，如果此时增加了描述信息，则会 **直接读出文本名字** ，但如果此时这是一个可以交互的按钮呢？

对于我们普通用户而言，大家知道这里可以点击，但是他们并不知道，所以在这里如果使用 `ImageButton` ,此时在无障碍下的反馈就是：

> xxxApp，返回 **按钮**。双击进入下一步

对于视障用户而言，这将提高他们的使用便利度,以方便他们的使用。

---

### 改造非标准组件的选中状态

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gynrjs3spvj30ia0a0jrl.jpg" alt="image-20220123180508158" style="zoom:33%;" />

类似上述的截图，如果这里的选择框使用的是 `ImageView` 去定义,此时无障碍服务将无法识别当前相应的状态。

如果使用系统默认的组件，如 `CheckBox` 或者 `Switch` ，则可以正确读出相应状态，如果因为业务等相关问题无法直接调整，可以通过手动添加无障碍代理的方式，间接的为控件增加无障碍下的状态，如下代码所示：

```kotlin
view.accessibilityDelegate = object : View.AccessibilityDelegate() {
            override fun onInitializeAccessibilityNodeInfo(
                host: View?,
                info: AccessibilityNodeInfo?
            ) {
                super.onInitializeAccessibilityNodeInfo(host, info)
                info?.isCheckable = true
                // 设置选择状态
                info?.isChecked = isSelect
            }
        }
```

---

### 增加按钮触摸范围

在MD的设计中，按钮的可触摸范围至少为 **48dpx48dp** ,所以如果我们的按钮大小不足，则可以使用下述方向进行优化：

- 使用 `padding` 为按钮图标周围增加填充
- 使用 `touchDelegate` ,具体详见 [读源码长知识 | 原来可以这样扩大 View 点击区域](https://juejin.cn/post/6968237652017414151)

### 处理焦点

对于部分 View ，我们可能并不想在无障碍下被读取，或者需要将一些view进行 `合并` ，此时就可以为其增加 `importantForAccessibility` 属性，默认提供了以下几个选项：

```xml
<attr name="importantForAccessibility" format="integer">
            <!--默认行为,系统根据其子view的类型自动判断是否对其进行读取-->
            <enum name="auto" value="0" />
            <!--允许在无障碍模式下访问-->
            <enum name="yes" value="1" />
            <!--禁止在无障碍模式下访问-->
            <enum name="no" value="2" />
            <!--表示其子view都禁止无障碍模式下访问-->
            <enum name="noHideDescendants" value="4" />
</attr>
```

**示例：**

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gymtv7xhugj30ic0c63z3.jpg" alt="image-20220122223943594" style="zoom:50%;" />

比如上述示例，如果没加特别描述，默认情况下，点击时，红框里只会有 **微信好友(TextView)** 在无障碍模式下触发，这显然不是我们希望的，此时就可以通过将 `contentDescription` 上移到外部红框的 `ViewGroup` 中。

相应的，某些业务规则下，如果并不想其在无障碍下被选中，比如 [微博] 此时如果没有安装,则可以 **忽略其焦点** 及 **禁用** 在无障碍下的可访问性:

```xml
android:focusable="false" 
android:focusableInTouchMode="false" 
android:importantForAccessibility="no"
```

```kotlin
view.isFocusable = false
view.isFocusableInTouchMode = false
view.importantForAccessibility = IMPORTANT_FOR_ACCESSIBILITY_NO
```

---

### 兼容自定义View

我们的项目中多多少少都会存在很多自定义View,有些是基于View,有些是基于 ViewGroup 的组合效果。那么如何对自定义View做兼容呢？

也非常简单，只需要增加相应的 `contentDescription` 即可，相应所有View页都可以去调用 **setContentDescription()** 。

需要注意的是，如果你自定义的 `View` 是一个 **按钮** ，而且没有使用 `ImageButton` 或者单独 `继承自Button` ,那么此时如果只是增加 `contentDescription` ，那么当view在无障碍下点击时，则只会读取`描述`，而使用了 `ImageButton` 或者 `Button` 的在无障碍模式下会被读作xx按钮，相比起来，后者更象征着这具有一个行为作用，而前者仅仅像一个普通文本，这对视障用户而言，体验将影响很多。

**所以我们要如何快速的兼容呢？**

其实很简单，如果你注意观察ImageButton与Image之间的区别，你就会发现？

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyklxo0x25j30ze0u00xs.jpg" alt="image-20220121003411236" style="zoom:33%;" />

**getAccessibilityClassName()**,我们只需要返回相应的 `Class Name` 即可。所以如果你的某个 View 具有 **行为** 作用，或者代表着是一个自定义的 **按钮** ，那么就可以重写其的这个方法，返回 `Button` ,或者 `ImageButton` ，这样在无障碍模式下，其就会被系统判断为是一个具有交互作用的按钮。

当然，严格意义上而言，我们应该尽可能使用系统组件，但业务的变化导致我们不可能一直如此，所以上述的方案也是一种比较取巧的方式。

更多关于自定义View的适配，可以查看[Android官方文档-让自定义视图使用起来更没有障碍](https://developer.android.com/guide/topics/ui/accessibility/custom-views#populate-events)，里面主要是讲了通过无障碍代理类来实现。

## 测试你的无障碍适配

相应的，Google 也提供了一些工具用于查看你的适配是否合理。

比如无障碍功能扫描仪，官方[使用文档](https://support.google.com/accessibility/android/faq/6376582?hl=zh-Hans)如下。

无障碍功能扫描仪主要用于对当前屏幕上所有的 `View` 进行扫描，并给出建议，主要包括以下方面：

- 内容标签
- 触目目标的尺寸
- 是否存在可点按的内容
- 文本和图片的对比度

我们可以用其作为一个参考作用来使用。

比如如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyns2o3bnaj317w0oq0y3.jpg" alt="image-20220123182316744" style="zoom:33%;" />

其会自动将一些认为可以优化的 `View` 标注出来，有些是触摸按钮太小，有些是对比度不够，在开发过程中，我们可以借此来实现快速调整。

## 后记

Hi，这里属于我个人的一些想法，想了一下还是觉得应该写出来，我觉得每个技术同学都热爱自己的用户，所以如果可以做到更好，我们一定不愿意做的差劲，虽然有时候碍于时间的繁忙，但大家还是更愿意尽量做的更好。

我是一个比较感性的人，第一次看到无障碍功能相关的反馈，是在几个月前，群里有同事发了一张截图，是16年时ios那边有用户反馈，建议适配一下无障碍。

![image-20220123103250833](https://tva1.sinaimg.cn/large/008i3skNly1gyneh6p41kj31rs0j2gsx.jpg)

当第一次看到这个的时候，有种悲伤的感觉并且也有点高兴。

作为一个开发者，我们的App被大多数人所使用，这是一种认同的感觉；相应的，也有点悲伤，我们其实可以做的更好。于是那一天，我发了这样一条朋友圈：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gynej21p0jj30sm0h6q5t.jpg" alt="image-20220123103438849" style="zoom:50%;" />

其实适配无障碍并不困难，**Android团队** 已经做了很多，而我们作为开发者，其实只需要稍微注意一下，就可以写出较好的无障碍体验。

技术应该有 **[深度]**，但也可以有 `[温度]`。

> 是谁来自山川湖海，却囿于昼夜、厨房与爱
>
> 《万能青年旅店乐队》

## 参考

- [随手记Android无障碍实践](https://juejin.cn/post/6844903609071566855)
- [Android无障碍适配准则](https://developer.android.com/guide/topics/ui/accessibility/apps)
- [让自定义的视图使用起来没有障碍](https://developer.android.com/guide/topics/ui/accessibility/custom-views)
- [Android无障碍功能帮助](https://support.google.com/accessibility/android/faq/6376582?hl=zh-Hans)



> 我是Petterp,一个三流开发，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！