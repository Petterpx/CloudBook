# Android无障碍代理的那些事

![科技与人文](https://tva1.sinaimg.cn/large/e6c9d24ely1h0n6az5zrgj20hs0bvdgf.jpg)

Hi,很高兴见到你！

> 本篇是无障碍系列第二篇 - Android无障碍代理的那些事

本篇将聊一聊什么是无障碍代理，及结合实际场景，分享一下我们对于无障碍代理的使用,并且如何让其更加易用。

## 什么是无障碍代理？

当用户在无障碍模式下时，用户在界面上与View的所有操作，都会触发相应的无障碍事件，这些事件最终由 无障碍服务处理 ，其会利用这些事件中的信息生成反馈和提示。自Android1.6(Api-4)开始，Android提供了相应的无障碍事件的代理扩展，我们可以通过实现相应的无障碍代理类(`AccessibilityDelegate` 或 `AccessibilityDelegateCompat`)，从而监听相应的方法回调，完成一些配置或者参数的更改，以便满足某些场景下的更改。

## 它可以做什么？

- 响应无障碍事件，完善自定义的无障碍提示,以及做出一些更改
- 捕获在用户无障碍下的行为信息,比如做数据统计或分析

> 前者这也是无障碍代理诞生的主因，而后者是我们近期在排查时，发现某个厂商sdk其内部的一个操作，也是有点惊讶。

## API详解

**AccessibilityDelegate**

具体Api解释如下,以下内容来源于 [Android文档](https://developer.android.com/guide/topics/ui/accessibility/custom-views?hl=zh-cn#accessibility-methods)：

### Api4

- `sendAccessibilityEvent()`

  当用户对视图执行操作时调用此方法。事件根据用户操作类型进行分类，如 `TYPE_VIEW_CLICKED`。除非您要创建自定义视图，否则通常无需实现此方法。

- `sendAccessibilityEventUnchecked()`

  如果发起调用的代码需要直接控制对设备上是否启用无障碍功能 (`AccessibilityManager.isEnabled()`) 进行检查，则使用此方法。如果您实现此方法，则无论实际的系统设置如何，您都必须像已启用无障碍功能那样执行调用。您通常不需要为自定义视图实现此方法。

- `dispatchPopulateAccessibilityEvent()`

  系统会在您的自定义视图生成无障碍事件时调用此方法。从 API 级别 14 开始，此方法的默认实现会为此视图调用 `onPopulateAccessibilityEvent()`，然后为此视图的每个子级调用 `dispatchPopulateAccessibilityEvent()` 方法。为了在早于 4.0（API 级别 14）的 Android 修订版上支持无障碍服务，您必须替换此方法并使用自定义视图的描述性文字填充 `getText()`，这些文字会由 TalkBack 等无障碍服务读出。

### Api14

- `onPopulateAccessibilityEvent()`

  此方法为您的视图设置 `AccessibilityEvent` 的文字转语音提示。如果该视图是生成无障碍事件的视图的子级，则也调用此方法。

  > **注意**：修改此方法中除文字之外的其他属性可能会替换其他方法设置的属性。虽然您可以使用此方法修改无障碍事件的属性，但应将这些更改限制为文字内容，并使用 `onInitializeAccessibilityEvent()` 方法修改事件的其他属性。
  >
  > **注意**：如果此事件的实现会完全替换输出文字且不允许布局的其他部分修改其内容，则请勿在您的代码中调用此方法的超类实现。

- `onInitializeAccessibilityEvent()`

  除了文字内容之外，系统还会调用此方法来获取有关视图状态的其他信息。如果您的自定义视图提供除了简单的 `TextView` 或 `Button` 之外的其他互动控件，则您应替换此方法并将有关视图的其他信息设置到使用此方法的事件中，如密码字段类型、复选框类型或者提供用户互动或反馈的状态。如果您替换此方法，则必须调用其超类实现，然后只修改超类未设置的属性。

- `onInitializeAccessibilityNodeInfo()`

  此方法为无障碍服务提供有关视图状态的信息。默认的 `View` 实现具有一组标准的视图属性，但如果您的自定义视图提供除了简单的 `TextView` 或 `Button` 之外的其他互动控件，则您应替换此方法并将有关视图的其他信息设置到由此方法处理的 `AccessibilityNodeInfo` 对象中。

- `onRequestSendAccessibilityEvent()`

  系统会在您的视图的子级生成 `AccessibilityEvent` 时调用此方法。通过此步骤，父视图可以使用其他信息修改无障碍事件。仅当您的自定义视图具有子视图且父视图可以向无障碍事件提供有助于无障碍服务的上下文信息时，才应实现此方法。

> 需要注意的是，如果我们的Api版本>=14,**即Android4.0及以上**，`则可以直接在View中实现上述方法`,
>
> 否则使用 **ViewCompat.setAccessibilityDelegate()** 或者 **View.setAccessibilityDelegate()** 设置相应的代理，从而重写相应的方法。

## 注意事项

无障碍代理有两种设置方式，默认的与兼容版本，即 `AccessibilityDelegate` 与 `AccessibilityDelegateCompat`。

> 加compat的一般都为前者的兼容版本,以满足低版本的一些功能兼容，但我还是 **强烈** 建议大家使用后者。

具体原因是：

使用 `AccessibilityDelegate` 作为代理类时，当我们将 `view.accessibilityDelegate=null` 时,即我们解绑代理时，我们认为这个代理之后不会被调用,实则它依然会每次被调用，比较离谱。

而当你使用 `AccessibilityDelegateCompat` 时,你会发现当你调用 `ViewCompat.setAccessibilityDelegate(view, null)` 时，你之前的代理类就不会被调用,是不是很离谱，而观察源码你会发现，当使用 `ViewCompat`设置为 `null` 时，内部不是直接赋值，而是给予了一个新的实例。

![image-20220321101044497](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ha5scm9fj21nw0rs78o.jpg)



## 让无障碍代理更易用

作为我们目前的业务，无障碍代理更多的场景是，为某个 **[没有状态]** 的 `View` 增加选中状态,于是我们能很轻松的写出以下代码:

```kotlin
val delegateCompat = object : AccessibilityDelegateCompat() {
    override fun onInitializeAccessibilityNodeInfo(
        host: View?,
        info: AccessibilityNodeInfoCompat?
    ) {
        super.onInitializeAccessibilityNodeInfo(host, info)
      	// 你的自定义逻辑
        info?.isChecked = xxx 
        info?.isCheckable = true
    }
}
ViewCompat.setAccessibilityDelegate(this, delegateCompat)
```

最简单的优化，我们自然可以把它提取出来，以便后续复用，于是就有了如下的代码：

### 优化1 ->

```kotlin
inline fun lazyAcesDeleteSelectSimple(crossinline obj: () -> Boolean): Lazy<AccessibilityDelegateCompat> =
    lazy {
        object : AccessibilityDelegateCompat() {
            override fun onInitializeAccessibilityNodeInfo(
                host: View?,
                info: AccessibilityNodeInfoCompat?
            ) {
                super.onInitializeAccessibilityNodeInfo(host, info)
                info?.isChecked = obj.invoke()
                info?.isCheckable = true
            }
        }
    }

//使用处
val xxxViewAcesDelegate by lazyAcesDeleteSelectSimple{
  //你的逻辑
   true or false
}
val view=View(null)
ViewCompat.setAccessibilityDelegate(view, xxxViewAcesDelegate)
```

这段代码也很好理解，我们借助 `Lazy` 委托,在后续使用时再初始化这个代理,并将方法进行了抽取。

### 优化2 ->

前段时间同事在review代码时提到，你的无障碍这块能不能再简化一点?

作为我们的业务场景，大多数情况下，增加代理 **只是为了给View或者ImageView增加一个选中状态**，我看你现在的写法是写了一个统一的调用方法和回调，其实已经挺好。那能不能更简化一点，比如我们未来其他的配置或者更改等等。

比如 `View` 自身的也有 `isSelected` 属性，你看看能不能做到只更改这个属性,就可以自动的适配无障碍下的选中状态。对于外部调用者而言，我无需去关心无障碍也能轻松适配。比如大家都知道 `contenDescortrion` 属性，但不一定人人都知道需要 **传递委托代理** ，复杂情况下还得重写相应方法，不够便捷。

![image-20220326124538093](https://tva1.sinaimg.cn/large/e6c9d24ely1h0n6qgdyjlj20js070aae.jpg)

听完之后，然后做了如下改良,思路如下：

- 增加无障碍接口,里面是一些 **[简化]** 的配置操作
- 继承自 `AccessibilityNodeInfoCompat` 并增加相应的回调函数，并实现上述无障碍接口
- 增加 `View` 的扩展属性,比如 `View.accessDelegate` , `View.isAccessSelected` ，前者返回无障碍接口,后者用于控制此 `view` 是否已选中。

示例代码如下:

#### 增加代理接口：

```kotlin
interface IAccessibilityDelegate {

    /** 无障碍下是否选中
     *
     * 默认会使用此字段来控制,如果实现了[setSelectedProvider],则此字段仅作为状态查看 */
    var isSelect: Boolean

    /** 使用回调的方式设置无障碍选中状态,某些业务场景下会用到,优先使用[isSelect]即可
     *
     * 注意:如果此方法被启用,则优先使用此回调,此时[isSelect]仅作为状态查看 */
    fun setSelectedProvider(obj: (() -> Boolean)?): IAccessibilityDelegate

    /** 此方法为无障碍服务提供有关视图状态的信息,增加此监听便于外部监听 */
    fun setInitializeNodeInfoListener(obj: (View, AccessibilityNodeInfoCompat) -> Unit): IAccessibilityDelegate

    /** 解绑所有回调 */
    fun unBind()
}
```

#### 增加代理实现类：

```kotlin
class CustomAccessibilityDelegateCompat : AccessibilityDelegateCompat(), IAccessibilityDelegate {

    override var isSelect: Boolean = false

    /**
     * 内部重写的一些方法,暂时只用到了这些,如果有其他的,可以加到下面,并更改[IAccessibilityDelegate]
     * -> */
    private var onSelectedProvider: (() -> Boolean)? = null
    private var onInitializeAccessibilityNodeInfo: ((View, AccessibilityNodeInfoCompat) -> Unit)? = null

    /** 此方法为无障碍服务提供有关视图状态的信息。 */
    override fun onInitializeAccessibilityNodeInfo(
        host: View?,
        info: AccessibilityNodeInfoCompat?
    ) {
        super.onInitializeAccessibilityNodeInfo(host, info)
        if (host == null || info == null) return
        onInitializeAccessibilityNodeInfo?.invoke(host, info)
        isSelect = onSelectedProvider?.invoke() ?: (isSelect || host.isSelected)
        info.isChecked = isSelect
        info.isCheckable = true
    }

  
    override fun setSelectedProvider(obj: (() -> Boolean)?): IAccessibilityDelegate {
        onSelectedProvider = obj
        return this
    }

    override fun setInitializeNodeInfoListener(obj: (View, AccessibilityNodeInfoCompat) -> Unit): IAccessibilityDelegate {
        onInitializeAccessibilityNodeInfo = obj
        return this
    }

    override fun unBind() {
        onInitializeAccessibilityNodeInfo = null
    }
}
```

#### 增加kt扩展类

```kotlin
@file:JvmName("AccessibilityUtils")

private const val ACCESS_DEFAULT_CONTENT_DESCRIPTION = "ACCESS_DEFAULT_CONTENT_DESCRIPTION"

/**
 * 设置当前View在无障碍下是否已选中
 * 使用view默认的isSelect也同样受用,前提是已经调用过[initXcfAccessDelegate]
 * */
var View.isAccessSelected: Boolean
    get() = accessDelegate.isSelect || isSelected
    set(value) {
        accessDelegate.isSelect = value
    }

/** 获取自定义的无障碍接口 */
val View.accessDelegate: IAccessibilityDelegate
    get() {
        val delegate = ViewCompat.getAccessibilityDelegate(this) as? IAccessibilityDelegate
        if (delegate == null) {
            val newDelegate = CustomAccessibilityDelegateCompat()
            ViewCompat.setAccessibilityDelegate(this, newDelegate)
            return newDelegate
        }
        return delegate
    }

/** 初始化无障碍委托,满足一些基础view的免操作适配 */
@JvmOverloads
fun View.initAccessDelegate(contentDescription: String = ACCESS_DEFAULT_CONTENT_DESCRIPTION):
    IAccessibilityDelegate {
    if (contentDescription != ACCESS_DEFAULT_CONTENT_DESCRIPTION)
        this.contentDescription = contentDescription
    return AccessDelegate
}
```

#### 使用方式：

```kotlin
// 例如有一个使用ImageView做开关的 [历史代码]
fun test() {
    val toggleView = ImageView(context)
    // 通过扩展属性设置
    toggleView.isAccessSelected = false
    // 通过自定的逻辑去设置
    toggleView.initXcfAccessDelegate("xx开关").setSelectedProvider(::checkToggle)
}

/** 你的业务逻辑 */
fun checkToggle(): Boolean = false
```

经过上述这样的步骤，我们就可以较为轻松的为任意 `View` 增加选中状态，同时对于其他同学而言，成本也比较低。

上述如果要增加新的api,也可以更改相应的代理类,同时在接口中定义新的 `setXXXListener` 即可。

## 总结

通过无障碍事件的重写，极大程度上减轻了我们在 `View` 上的适配成本，对于无法直接重写相应方法的，我们也可以间接通过无障碍代理去完成，相对来讲成本并不高，再加上我们对其进行相应的封装后，使用难度就更加容易。

### 参考

- [让自定义视图使用起来更没有障碍](https://developer.android.com/guide/topics/ui/accessibility/custom-views?hl=zh-cn#accessibility-methods)

> 我是Petterp,一个三流开发，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！



