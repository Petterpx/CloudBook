# 如何写一个Compose状态页组件 (修正篇)

在上个月前，我写了这样的一篇文章，[开源 | 如何写一个好用的 JetPack Compose 状态页组件](https://juejin.cn/post/7026641600067420190) 。里面讲了如何去写一个 `compose` 状态页组件，结果这反而是错误的开始，本篇就是对上述的一个修正及反思过程。

## 反思

在上篇中，我简单实现一个 `compose` 中的状态页，但为了解决重组后造成的重新加载问题，当时没有想到该更好的如何处理这个问题，于是采用了命令式的方式去操纵实现了整个流程，这与 `compose` 的声明式明显格格不入。

旧的整体流程如下所示：

![image-20211104155419940](https://tva1.sinaimg.cn/large/008i3skNly1gxa5oujuf9j31k80u00w6.jpg)

在 `viewModel` 中定义了一个当前状态,并且定义了加载数据的方法, 在Ui部分，我使用了一个 `rememberState` 这个方法缓存当前的 `state` 状态,在这里方法中我们还可以初始化 `state` 的部分回调，并且启用了加载数据，这将触发 `onRefresh` 回调，即加载页面数据，从而调用了我们 `ViewModel` 内部的 **getData()** 方法，当数据加载完成，我们便可以直接驱动这个 `state` 展现当前加载成功状态，从而触发外部的重组，于是我们的 `StateCompose` 将展示成功页面。

**这样的实现可以吗？**

勉强可以，但是其不符合 `compose` 的设计，并且需要和 `viewModel` 关联，整体更像是命令式的驱动，虽然内部利用了 **_state** 的改变从而引发组件的重组，但整个过程仍然像一个蹩脚老头。

**而当时的我，在写完文章后，还兴冲冲的投稿到了郭大的公众号，在此对看过本篇的同学先说一声抱歉，因为我个人的学艺不精而导致错误的思想传递。**

也特别感谢以下同学的指正：

> 郭大公众号 - **NullPointerException**

![image-20211211202432365](https://tva1.sinaimg.cn/large/008i3skNly1gxa5xkr38ij31dm06i75e.jpg)

> 掘金 - **[FunnySaltyFish](https://juejin.cn/user/2673613109214333)** ，[相关文章链接](https://juejin.cn/post/7033755169213022216)

![image-20211211202457416](https://tva1.sinaimg.cn/large/008i3skNly1gxa5y1cugtj314o0as0uo.jpg)

当看到其他同学的反馈后，深感愧疚，于是去官网看了下相关文档，重新整理下实现。

在此先对上述同学表示感谢。



## 使用 LaunchedEffect 实现

在开始我的想法之前，先解释一下上面同学提到的 [LaunchedEffect](https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn#launchedeffect) 以及 [produceState](https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn#producestate)。

![image-20211215223731461](https://tva1.sinaimg.cn/large/008i3skNly1gxew97xwloj32i80m8te3.jpg)

`LaunchedEffect` 用于在某个可组合项的作用域内运行挂起函数，其是没有返回值的，主要适用于在可组合项内执行一段挂起函数。

`produceState` 则更多是用于将一段非 `compose` 的代码状态转换为具有 `compose` 状态，即其附带了返回值State。

从具体的实现上，我们不难发现，`produceState` 其内部也是使用了 `LaunchedEffect` ,不同的是，其相比 `LaunchedEffect` 增加了一个`initalValue`，并且使用了 `remember` 缓存了这个 **value** ,当其改变后，从而触发外部使用者的重组，当然我们也可以传递一个 **key** 进来，从而当 **key** 改变后，触发 `LaunchEdEffect` 的重新执行，而我们就可以将刷新的一些工作放在其附带的挂起函数里中，这也就避免了我之前一直考虑的如何解决重组后所导致的没必要初始化的 [副作用] 。

按照这个流程，我们可以写出下面的状态页示例代码：来自 **FunnySaltyFish** 同学：

```kotlin
sealed class PageData<out T> {
    data class Success<T>(val t: T? = null) : PageData<T>()
    data class Error(
        val throwable: Throwable? = null,
        val value: Any? = null
    ) : PageData<Nothing>()

    object Loading : PageData<Nothing>()
    data class Empty(val value: Any? = null) : PageData<Nothing>()
}

@Composable
fun <T> ComposeProducerState(modifier: Modifier, obj: suspend () -> T) {
  	var key by remember(key) {
        mutableStateOf(true)
    }
    val state by produceState<PageData<T>>(initialValue = PageData.Loading, key) {
            value = try {
                PageData.Success(obj())
            } catch (e: Throwable) {
                PageData.Error(e)
            }
    }

    Box(modifier = modifier) {
        when (state) {
            is PageData.Success -> {}
            is PageData.Loading -> {}
            is PageData.Error -> {
                // ...
              	// 点击后执行
                key = !key
                // ...
            }
            is PageData.Empty -> {}
        }
    }
}
```

> 总体上思路清晰，我们在缓存了一个 `key`, 为什么要缓存这个 `key`,主要是为了在其改变后，触发 `produceState` 内部 `LaunchedEffect` 的执行。
>
> 接下来我们将其将这个 `key` 传递给了 `produceState`，以避免 **ComposeProducerState** 方法 重组时，`produceState` 内部 `LaunchedEffect` 被迫导致的重复执行，然后在其的挂起函数中，我们执行了获取数据方法，从而设置了状态页的初始化。
>
> 而下面的 `Box` 代码里，当加载页处于 `Error` 时，我们只需要改变 **key** ，从而引起 `produceState` 的重组，接着就又会触发我们的数据加载方法。从而就可以简单实现一个状态页，当然具体的逻辑我们可以任意去改。

## 优化，如何能更实用

在 `compose` 中，状态的改变其实我们都应该考虑到是否会对其他组件造成不必要的重组影响，所以 `compose` 中我们应该尽量保证每个组件都 **保持独立** 。但相应的，有些时候我们也需要由外部传递状态进来。

回到上述的实现中，上述方式虽然可以实现，但是不够灵活，比如我们可能还需要将状态提升出去，以便让外部在重组时可以知道当前是什么状态，或者说，我们希望状态由外部自行维护。于是针对此，我们应该怎么做？

我们先看一下通用的设计思路，`LazyColumn` 就相当于 **Android** 中的 `RecyclerView` ,而我们如果要监听 `LazyColumn` 列表当前状态时，就需要手动传递一个 `state` 进去,从而以便于重组时我们可以实时拿到当前列表状态。默认是使用的 **rememberLazyListState()** ，具体源码如下：

![image-20211214214119653](https://tva1.sinaimg.cn/large/008i3skNly1gxdp0ggnlkj31o10u07bd.jpg)

而 `ComposeState` 也正是需要这样的一个实现，借此，所以我们可以定一个通用的状态管理类，其目的就是保存当前的状态，以便用户在外访问当前状态，维护状态，从而将状态提升到调用处，当用户外部不需要这个状态时，我们默认实现一个即可，具体如下所示：

```kotlin
/** 页面状态 */
class PageState<T>(state: PageData<T>) {

    /** 内部交互的状态 */
    internal var interactionState by mutableStateOf(state)

    /** 供外部获取当前页面数据状态 */
    val state: PageData<T>
        get() = interactionState

    val isLoading: Boolean
        get() = interactionState is PageData.Loading
}
```

如上所示，内部有一个 `interactionState` 的字段，其代表了我们当前的状态，当其改变后，从而触发相应使用处的重组。

之所以 `interactionState` 要使用 **internal** ， 是因为在 `compose` 中，我们不想写成传统命令式的操作，即我们不应该让用户可以直接调用到此字段，对于状态的更改，我们希望只存在单向的方式，也就是说这个 `PageState` 组件只允许读取，并不提供随意的更改内部变量，以此避免可能带来的状态问题。

对于外部访问而言，我们提供了 `state` ，这样调用者就可以在重组时知道当前最新是什么状态，从而做一些特定的操作，当然我们也可以提供一些额外的快捷字段，比如 `isLoading` 字段,判断当前是否处于加载中等等。

在做好了上述的组件之后，我们重新来设计一下我们的 `ComposeState` 。

具体如下所示：

```kotlin
// 用于缓存页面状态
@Composable
fun <T> rememberPageState(state: PageData<T> = PageData.Loading): PageState<T> {
    return rememberSaveable {
        PageState(state)
    }
}

@Composable
fun <T> ComposeState(
    modifier: Modifier,
    pageState: PageState<T> = rememberPageState(),
    loading: suspend () -> PageData<T>,
    loadingComponentBlock: @Composable (BoxScope.() -> Unit)?,
    emptyComponentBlock: @Composable (BoxScope.(PageData.Empty) -> Unit)?,
    errorComponentBlock: @Composable (BoxScope.(PageData.Error) -> Unit)?,
    contentComponentBlock: @Composable (BoxScope.(PageData.Success<T>) -> Unit)
) {
    val scope = rememberCoroutineScope()
    Box(modifier = modifier) {
        when (pageState.interactionState) {
            is PageData.Success -> contentComponentBlock(pageState.interactionState as PageData.Success<T>)
            is PageData.Loading -> {
                loadingComponentBlock?.invoke(this)
                scope.launch {
                    pageState.interactionState = loading.invoke()
                }
            }
            is PageData.Error -> StateBoxCompose({
                pageState.interactionState = PageData.Loading
            }) {
                errorComponentBlock?.invoke(this, pageState.interactionState as PageData.Error)
            }
            is PageData.Empty -> emptyComponentBlock?.invoke(
                this,
                pageState.interactionState as PageData.Empty
            )
        }
    }
}

@Composable
fun StateBoxCompose(block: () -> Unit, content: @Composable BoxScope.() -> Unit) {
    Box(
        Modifier.clickable {
            block.invoke()
        },
        content = content
    )
}
```

如上所示，我们定义了一个名为 **rememberPageState()** 的方法，用于缓存 `PageState` 当前状态，并且 状态页组件 `ComposeState` 需要接收一个 `pageState` 对象，默认我们使用 **rememberPageState()** 实现,由 `ComposeState` 组件 自己管理状态。

当然用户在外部自己维护状态,然后将其传递进来。

在 **loading()** 回调里，其代表的是刷新的功能，当调用时，用户需要手动返回当前得到的状态，这样我们就将具体的业务逻辑交给了用户，至于究竟会是错误还是正确，还是null页面,让用户自己做决定，而组件只负责展示逻辑。

具体优化后的代码见这里：

[StateCompose](https://github.com/Petterpx/StateX/blob/main/compose/src/main/java/com/petterp/statex/compose/StateCompose.kt)

[demo示例见这里](https://github.com/Petterpx/StateX/blob/main/app/src/main/java/com/petterp/statex/app/ComposeStateActivity.kt)

> ps:在写完这里后，我看了下前辈 RicardoMJiang 的 [StateLayout](https://github.com/shenzhen2017/ComposeStateLayout/blob/main/statelayout/src/main/java/com/zj/statelayout/ComposeStateLayout.kt) 示例，发现实现上比较相似，看来大佬两个月前就这样实现了，这里再次向前辈投去崇拜。

## 总结

本篇中涉及到的一些 `Compose` 的概念：

- [副作用的处理](https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn)
- [重启效应](https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn#restarting-effects)
- [状态提升](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-hoisting)

最后，非常感谢以上反馈过的同学。本篇拖了挺久，向看过上一篇的同学再说一声抱歉。

在本篇，我们从传统命令式的视角切回到了声明式实现思路，重新实现了一个 `Compose` 中的状态页组件，具体实现与细节大家可以看 [上述源码](https://github.com/Petterpx/StateX/blob/main/compose/src/main/java/com/petterp/statex/compose/StateCompose.kt)，也可以也可以根据自身业务进行更改。初学者也能借助不同思路实现之间的差别感受一下思路的转换，比如我。

如果还有其他问题，大家也可以进行反馈。

最后，我的小猫皮卡丘结尾。大家加油！

![image-20211216093401048](https://tva1.sinaimg.cn/large/008i3skNly1gxff89mcwtj317m0u0tf1.jpg)
