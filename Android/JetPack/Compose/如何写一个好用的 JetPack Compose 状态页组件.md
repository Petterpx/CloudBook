# 开源 | 如何写一个好用的 JetPack Compose 状态页组件

## Hi, :)

世界很大，也很小，组件很多，也很少。

关于开发中常见的状态页组件，我们已经见了很多，但是在 `JetPack Compose` 中该如何去写呢？虽然也有大佬写了相关demo,但是如果要应用到实际中，不免有些捉襟见肘。

本篇要解决的就是在 `JetPack Compose` 中如何定制一个符合实际业务场景的状态页工具，并且分析具体原理与设计思路。

## 效果图



<div align=center><img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d813fc28137c4718851e916aaf67e0b6~tplv-k3u1fbpfcp-zoom-1.image" alt="statex_simple" width="240" height="470" /></div>

这个效果图很简单，就是普通的一个状态页，所以也没什么值得说的，我们接下来分析一下，如果要实现一个状态页组件，需要有哪些基础功能。

## 需求分析

- 支持 `compose` 与 `view`
- 分层设计，按需引入
- 支持全局/局部配置默认缺省页
- 支持全局重试与防抖处理
- ...

看完基本条件，其实也都不难，在 `View` 中设计一个状态页组件，大家都知道怎么做，但是 `Compose` 呢？ 那么我们下面就开始构思一下，如何设计这个状态页组件。

## 基本思路

其实只要写过 `compose` 的代码，应该都明白，其实更简单了。因为 `compose` 是声明式的编程思想，即我们可以理解为数据驱动，所以最简单的做法：

> 定义一个变量，然后每次更改这个变量，变量改变之后，相应的使用这个变量的地方就会触发重组，于是我们可以随手写出下面的伪代码：

```kotlin
   val state = mutableStateOf (Loading)

   when(state){

     Loading -> {}

      Error -> {}
  
      Content -> {
       //加载错误了, 更改状态即可
       state = Error
     }

      xxx

 }
```

没错，在 `compose` 中实现就是这么简单，原理也很好理解。

---

### 不足之处

但如果你真的这样去写了，你可能已经进入一个圈套？试想一下，这个真的符合我们实际业务场景吗？

我们先还原一个真实的业务场景。

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f44e020953e046b68427537da84f443f~tplv-k3u1fbpfcp-zoom-1.image" alt="image-20211103200037374 " style="zoom:33%;" />

这是一个展示用户点赞排行榜的列表页，按照我们常规的思路，我们会怎么写：

> 1. **先展示loading**
> 2. **请求数据**
> 3. **请求成功-设置数据，错误-显示缺省页**

这个思路没有问题，在传统 `view` 中我们一般都是这样实现，但是 `compose` 中呢，我们按照上面的思路写一个伪代码。

```kotlin
@Composable
fun Test() {
    var state = remember {
        mutableStateOf(StateEnum.LOADING)
    }
    when (state.value) {
        StateEnum.LOADING -> {
        }
        StateEnum.CONTENT -> {
            // 展示成功
        }
        StateEnum.ERROR -> {
            // 展示错误
        }
    }
    // 获取结果
    val data = getData()
    if (data is Success) {
        state.value = StateEnum.CONTENT
    } else if (data is Error) {
        state.value = StateEnum.ERROR
    }
}
```

这个流程对吗？如果真这样写，那么恭喜你，你已经陷入了老路子，代码也将死循环。

> 成也 **重组** ，败也 重组 ,传统的 `view` 中，属于命令回调式，因为相应的方法只会在命令时执行，我们不必担心无关方法被调用。而在 `compose` 中，重组会执行所有调用的地方，并判断是否需要执行，我们必须要考虑如何避免重复的重组。

所以如果上述改变 `state` 后，接下来还会继续执行 **getData()** ，那么该怎么做呢？

---

### 如何解决?

你可能会想，既然如此，那我直接在 **CONTENT** 中写请求逻辑不就行吗？

> 可以，但是问题来了，那 **Loading** 还怎么展示？

那我直接去 **Loading** 中触发请求逻辑？

> 可以做，但是怎么做呢？虽然我知道这样能做，但是具体该怎么封装好呢？
>
> 于是有没有一个简便的，封装好的组件供我参考或者拿来就用呢？

<div align=center><img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4a90ae41e694c43bab5c68018bec17b~tplv-k3u1fbpfcp-zoom-1.image" alt="image-20211103200037374 " width="500" height="300" /></div>

为了解决上述问题，我写了一个简单组件 [StateX](https://github.com/Petterpx/StateX) ，大家可以自行copy更改，下面开始分析一下设计思路。

## 解析 StateX

要设计一个可以供 `compose` 与 `View` 都可以使用的组件，不可避免的就需要两个model,分层去设计，并且支持按需引入，对于共有的模块，还需要单独提到基础组件里，于是 `StateX` 分为三个模块：

- basic 基础层，放了一些compose与view共用的基础配置
- compose 属于compose的单独model
- view 属于view层的单独model

> 感谢 @掘金-[Range](https://juejin.cn/user/3034307821573672/posts)(业内俗称东哥)的 [StateLayout](https://github.com/liangjingkanji/StateLayout),view部分的核心代码来自这里，原因足够简单易用。

---

### 基础层-Basic 设计

既然要支持 `compose` 与 `View` ，那么基础需要哪些功能呢？

```kotlin
enum class StateEnum {
    LOADING,
    EMPTY,
    ERROR,
    CONTENT
}
```

```kotlin
interface IState {
    val state: StateEnum
    var enableNullRetry: Boolean
    var enableErrorRetry: Boolean
		
  	/** 显示加载成功
     * @param [tag] 可以传递任意数据,会在回调处收到
     * */
    fun showContent(tag: Any? = null)
    ...
}
```

我们定义了一个基础接口,其代表了 `compose` 与 `view` 公用的接口, `StateEnum` 代表了对应的状态枚举。

**但是 compose 与 view 的配置项怎么设置呢？** 

> 因为两者的配置肯定不同，那么有没有一种方式也能统一这两者的设置。

为了便于设置，我定义了一个 `StateX` 的静态类。

```kotlin
object StateX {
    /** 默认点击防抖时间 */
    var defaultClickTime = 600L

    /** 空数据重试开关 */
    var enableNullRetry = true

    /** 异常重试开关 */
    var enableErrorRetry = true
}
```

乍一看好像并没有什么，这个静态类只是对应了一些基本的共用配置项，和其他model的配置项似乎关联不大。但是 `Kotlin` 支持扩展函数与方法，这样，通过唯一的 **StateX** 入口，我们便可以在相应的 `compose` 与 `view` 的model中增加基于 **StateX** 的扩展函数，便于增加配置项。就是这么简单。

---

### compose层设计

#### 配置设计

配置层是一个简单的类，同时我们定义了一个 `internal` 修饰的静态 **StateComposeConfig** 对象，以便组件内部访问,同时定义了 **StateX** 的扩展函数 **composeConfig** ,从而完成对 `compose-config` 的初始化，是不是比较简单。

```kotlin
class StateComposeConfig {
    ...
    internal var emptyComponent: stateComponentBlock = {}
    ...
    internal var onContent: stateBlock? = null
    ...

    fun onContent(block: stateBlock) {
        this.onContent = block
    }
		...

    fun emptyComponent(component: stateComponentBlock) {
        this.emptyComponent = component
    }
}

/** 内部使用的StateCompose配置 */
internal val composeConfig by lazy(LazyThreadSafetyMode.PUBLICATION) {
    StateComposeConfig()
}

/** 配置state-compose的配置 */
fun StateX.composeConfig(config: StateComposeConfig.() -> Unit) {
    composeConfig.apply(config)
}
```

---

#### 接口设计

相应的接口这里，我们需要 `compose` 也能感知到加载 `失败`，`错误`，`成功`，`loading`,同时附带了当前状态所对应的 `value` 。

```kotlin
interface IStateCompose : IState {

    /** 当前state附带的value */
    val tag: Any?

    /** 错误时的回调 */
    fun onError(block: stateBlock)

    ...
}
```

---

#### 具体实现类

具体的实现类 **StateComposeImpl** 也是非常简单简洁，我们在内部保留了一个 `_internalState` 变量，其代表当前状态，并且使用 `State` 包装，这样当我们调用 **showXxx()** 方法显示具体状态时，我们内部就会对相应的状态以及附带的 `value` 进行更新，从而 `_internalState` 就会更新，然后触发调用处的重组。

之所以要保留一个 `tag` ,是因为在实际中，我们一般在显示错误页面时，相应的文案都是根据具体错误更新，而非一成不变，所以需要缓存一个当前状态所对应的 `tag` ,这样便于我们在重组时使用。

```kotlin
class StateComposeImpl constructor(stateEnum: StateEnum = StateEnum.CONTENT) : IStateCompose {
		
  	// 这里是一个类型别名,只是为了省去方法参数中多余的写法，
  	// 坏处就是可能会降低可读性，具体根据自身而定
  	// internal typealias stateBlock = (tag: Any?) -> Unit
  	// 刷新时的回调，可以在这里回调里做数据加载，加载完成后调用showContent即可。
    private var onRefresh: stateBlock? = null
  	// 异常回调,默认使用的全局错误回调
    private var onError: stateBlock? = composeConfig.onError
    ...

    /** 当前内部可变状态 */
    private var _internalState by mutableStateOf(StateEnum.CONTENT)

    /** 当前状态内部缓存的tag */
    private var _internalTag: Any?

    override val state: StateEnum
        get() = _internalState
    override val tag: Any?
        get() = _internalTag

    override fun onError(block: stateBlock) {
        this.onError = block
    }
  	...

    override fun showError(tag: Any?) {
        onError?.invoke(tag)
        newState(StateEnum.ERROR, tag)
    }

    ...

    private fun newState(newState: StateEnum, tag: Any?) {
        _internalState = newState
        _internalTag = tag
    }
}
```

---

#### StateCompose

**StateCompose** 就是我们对外提供的一个具体 `Compose` 组件,外部只需要传入相应的控制器,同时也可以重写相应的状态对应的 `component` ，默认使用的是全局定义的。另外，我们在 `Error` 回调里对错误进行了防抖处理，并且在重试时会调用 **showLoading()** 方法，从而触发 `onRefresh` 的回调 刷新。

```kotlin
@Composable
fun StateCompose(
    stateControl: IStateCompose,
    loadingComponentBlock: stateComponentBlock 
  		= composeConfig.loadingComponent,
    ...
    contentComponentBlock: stateComponentBlock,
) {
    when (stateControl.state) {
        StateEnum.LOADING ->
      			loadingComponentBlock(stateControl, stateControl.tag)
        StateEnum.CONTENT ->
      			contentComponentBlock(stateControl, stateControl.tag)
      	StateEnum.ERROR ->
      			if (stateControl.enableErrorRetry) {
            StateBoxComposeClick(block = {
                stateControl.showLoading(null)
            }) {
                errorComponentBlock(stateControl, stateControl.tag)
            }
        } else errorComponentBlock(stateControl, stateControl.tag)
        ...
    }
}
```

---

#### 扩展工具

为了便于更好的解决实际存在的问题，直接在 **ui** 中解决不了，那么我们就拉上 `viewModel` ，为此提供了以下扩展便于使用：

```kotlin
/** 在ViewModel中生成一个 IStateCompose
 * @param stateEnum 默认的状态
 * */
inline fun ViewModel.lazyState(
    stateEnum: StateEnum = StateEnum.CONTENT,
    crossinline obj: StateComposeImpl.() -> Unit = {}
): Lazy<IStateCompose> = lazy(LazyThreadSafetyMode.PUBLICATION) {
    StateComposeImpl(stateEnum).apply(obj)
}

/**
 * 当state在ViewModel中缓存时,可以使用这个方法便于对state做初始化相关
 * 这样的好处就是可以将唯一初始化的东西放在这个 [block] 回调中,而不用担心重复初始化
 * @param composeState 要记住的状态State
 * */
@Composable
inline fun rememberState(
    composeState: IStateCompose,
    crossinline block: IStateCompose.() -> Unit = {}
): IStateCompose = currentComposer.cache(false) {
    composeState.apply(block)
}


/**
 * 记录state的状态,直接生成一个新的IStateCompose
 * @param stateEnum 默认的状态
 * @param block 对于IStateCompose的回调使用
 * */
@Composable
inline fun rememberState(
    stateEnum: StateEnum = StateEnum.CONTENT,
    crossinline block: IStateCompose.() -> Unit = {}
): IStateCompose = currentComposer.cache(false) {
    StateComposeImpl(stateEnum).apply(block)
}
```

### 使用方式

![image-20211104155419940](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04da7c9755ad4c62a5a199801f7a562f~tplv-k3u1fbpfcp-zoom-1.image)

如图所示，我们在 `viewModel` 中定义了一个当前状态,并且定义了加载数据的方法, 在Ui部分，我们使用了一个 `rememberState` 这个方法缓存当前的 `state` 状态,在这里方法中我们还可以初始化 `state` 的部分回调，并且启用了加载数据，这将触发 `onRefresh` 回调，即加载页面数据，从而调用了我们 `ViewModel` 内部的 **getData()** 方法，当数据加载完成，我们便可以直接驱动这个 `state` 展现当前加载成功状态，从而触发外部的重组，于是我们的 `StateCompose` 将展示成功页面。

> 小彩蛋：
>
> 为了满足有些时候我们可能不想在 `viewModel` 中管理状态，我也提供了另一个扩展 `rememberState`。
>
> 从而缓存一个 `IStateCompose` 的状态，但是这种场景实则不多，所以根据自身业务而定吧。

一切就是这么简单，在 `compose` 中如何使用状态页，已经分享大家了，至于大家要怎么改，可以参考 [StateX](https://github.com/Petterpx/StateX) 。

至于 `view` 部分的设计，大家一看源码就可以知道，并且大家已经 `view` 使用了多年，这个也不是本篇要讲的重点。

## 总结

本篇是 `Compose` 落地实践中比较常见的一篇，借此实践便于大家更好的理解 `Compose` 的编程思想。后续我将继续深追 `Compose` 的部分源码设计以及在实际落地中的场景解决方案。

如果本文对你有所帮助，欢迎点赞支持一下，大家加油 :)

## 感谢

[【开源项目】简单易用的Compose版StateLayout,了解一下~](https://juejin.cn/post/7010382907084636168)

[Android 一行代码构建整个应用的缺省页](https://github.com/liangjingkanji/StateLayout)

