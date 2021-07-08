# JetPack-Compose 实现原理解析



## 引言

Compose 即将迎来第一个正式版本，借此机会，本文主要分析一下 Compose 整个流程的底层实现，希望可以帮助大家更好理解，大家加油。

> 提示：本文大部分是 源码解读，虽然笔者画了详细的流程图，但依然存在一定晦涩，如果暂时不太理解，可以先收藏，后续想到了再慢慢看。

学完本文可以帮你解开什么问题？

- 为什么Compose 无需在意view层级问题，怎样嵌套都行？ (10s就能明白为什么)
- Compose 在底层的实现原理 (需要花点时间)



## 门外汉-从布局窥一眼

这是一段Compose的简单代码，我们演示了多层嵌套下的示例:

![image-20210703184658154](https://tva1.sinaimg.cn/large/008i3skNly1gs3ycfxhx8j31ke0u0ng0.jpg)

如果按照传统View的思维，我们不难发现，当前content(R.id.content(FrameLayout)->)布局中存在5层嵌套,这是极不可取的一种做法。

但是现在是Compose,最终的绘制真的会有5层吗？

我们打开Filpper看一下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gs3ynr82wrj30vg0g6tce.jpg" alt="image-20210703185751007" style="zoom:50%;" />

显然 R.id.content 下只有一个ComposeView，然后内部包含了一个 AndroidComposeView ，我们上述中的 Box 最终都被解析并安装到了这个自定义view上。

所以我们简单点可以总结为：

JetPack-Compose 其自定义了一个 基础容器-ComposeView，以及其他扩展View，比如AndroidComposeView ,并对其进行封装，对外提供了各种我们在上层所使用的各种组件或者容器。

所以当我们在 Compose中 setContent后，其初始化了一个ComposeView，并且添加了一个AndroidComposeView ,其承载了我们代码中所写的全部组件，并进行解析，最终绘制在了传统UI中。

**小彩蛋：**

为什么叫 AndroidComposeView 呢？

> Compose 现在不仅仅支持Android，现在预览版也支持Desktop ,所以很可能 ComposeView 很可能还会涉及其他UI系统。所以最顶层是ComposeView,而对于Android的支持为AndroidComposeView。
>
> 当然上述只是我一个猜测，如果存在错误，也属实正常。





## 初学 - setContent内部实现

我们在上面知道了Compose最终在Android UI的展现形式，那么它到底是怎样设置上去的呢，接下来我们就先上一道开胃小菜看看：

### ComponentActivity 

#### setContent

```kotlin
public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
) {
  	
  	//获取decorView->R.id.content->内第一个view,在compose中，默认为composeView
    val existingComposeView = window.decorView
        .findViewById<ViewGroup>(android.R.id.content)
        .getChildAt(0) as? ComposeView
  
		👉 -> 如果view不为null,则证明已经安装过，则重新设置内容
    if (existingComposeView != null) with(existingComposeView) {
      	//设置父Compose内容
        setParentCompositionContext(parent)
      	//设置当前页面内容
        setContent(content)
      	
    👉 -> 如果上面获取的view是空，也就是现在还没有安装，则生成一个新的去安装到 R.id.content 上
    } else ComposeView(this).apply {
      	//设置父Compose内容
        setParentCompositionContext(parent)
      	//设置当前页面内容
        setContent(content)
      	//设置TreeLifecycleOwner,实际上是修复Appcompat1.3及以下的bug，忽视即可
        setOwners()
      	//设置内容视图,activity的方法
        setContentView(this, DefaultActivityContentLayoutParams)
    }
}
```

从上面的源码，我们的下一步主要关注点为： 

- setParentCompositionContext()
- setContent()



### ComposeView

#### setParentCompositionContext

设置视图合成的父级context，里面仅仅是一个赋值，暂时跳过

```kotlin
fun setParentCompositionContext(parent: CompositionContext?) {
        parentContext = parent
}
```



#### setContent

//设置 compose UI内容，当视图被添加到窗口时调用。

```kotlin
fun setContent(content: @Composable () -> Unit) {
    shouldCreateCompositionOnAttachedToWindow = true
    this.content.value = content
  	//如果已经挂接到窗口，即view完全显示时，isAttachedToWindow=true
  	//第一次调用时，这里为false,所以当onAttachToWindows调用时，下面的方法才会被调用
    if (isAttachedToWindow) {
        createComposition()
    }
}
```

我们接下来去看看 AbstractComposeView 的  onAttachToWindows() 方法



### AbstractComposeView 

#### onAttachToWindows()

```kotlin
 override fun onAttachedToWindow() {
        super.onAttachedToWindow()
		
        previousAttachedWindowToken = windowToken
				
   			//这里当onAttached时为true,因为上一步已经赋值为true
        if (shouldCreateCompositionOnAttachedToWindow) {
            ensureCompositionCreated()
        }
    }
```



#### ensureCompositionCreated()

```kotlin
private fun ensureCompositionCreated() {
  	//整个流程中，我们没看见composition的初始化，所以这里为null
    if (composition == null) {
        try {
          	👉 1.  ///结合步骤1来看，就是一个用于判断是否add过的变量
            creatingComposition = true
            👉 2. // 先看 resolveParentCompositionContext() -> 得到一个顶级CompositionContext
          	
          	👉 3. // 拿着parentContext，传入setContent方法
            composition = setContent(resolveParentCompositionContext()) {
              Content()
            }
        } finally {
            creatingComposition = false
        }
    }
}
```

接下来我们接着上面继续往细了看具体实现。

#### resolveParentCompositionContext()

其作用是解析父组合上下文

```kotlin
private fun resolveParentCompositionContext() = parentContext
		//👉1. 
    ?: findViewTreeCompositionContext()?.also { cachedViewTreeCompositionContext = it }
    ?: cachedViewTreeCompositionContext
		// 👉2. 
    ?: windowRecomposer.also { cachedViewTreeCompositionContext = it }
```

##### 1. findViewTreeCompositionContext()

> 查找当前view树的父context
>
> ```kotlin
> fun View.findViewTreeCompositionContext(): CompositionContext? { 
>   	//先取自己的compositionContenxt，如果取到直接返回，否则不断向上找父级的context.
>     var found: CompositionContext? = compositionContext
>     if (found != null) return found
>     var parent: ViewParent? = parent
>     while (found == null && parent is View) {
>         found = parent.compositionContext
>         parent = parent.getParent()
>     }
>     return found
> }
> ```
>
> ##### View.compositionContext
>
> `compositionContext` 是一个扩展函数，内部使用tag保存当前context
>
> ```kotlin
> var View.compositionContext: CompositionContext?
>     get() = getTag(R.id.androidx_compose_ui_view_composition_context) as? CompositionContext
>     set(value) {
>         setTag(R.id.androidx_compose_ui_view_composition_context, value)
>     }
> ```

##### 2.View.windowRecomposer

我们接着上面的流程继续分析：

```kotlin
@OptIn(InternalComposeUiApi::class)
internal val View.windowRecomposer: Recomposer
    get() {
        ...
      	//这个rootView就是R.id.content对应的FrameLayout的第一个子view,即ComposeView
        val rootView = contentChild
      	// 因为上述是初始化状态，所以 rootParentRef 第一次也为null,所以我们继续往下看
        return when (val rootParentRef = rootView.compositionContext) {
          	//调用窗口Recomposer创建一个新的，并且把根content传入进去
            null -> WindowRecomposerPolicy.createAndInstallWindowRecomposer(rootView)
            ...
        }
    }
```

> #### View.contentChild 
>
> ```kotlin
> private val View.contentChild: View
>     get() {
>         var self: View = this
>         var parent: ViewParent? = self.parent
>         while (parent is View) {
>             if (parent.id == android.R.id.content) return self
>             self = parent
>             parent = self.parent
>         }
>         return self
>     }
> ```

###### createAndInstallWindowRecomposer(rootView)

创建一个 `Recomposer`，并且将其赋值给rootView的扩展变量 `compositionContext`

```kotlin
internal fun createAndInstallWindowRecomposer(rootView: View): Recomposer {
  			//使用默认的工厂创建一个Recomposer
        val newRecomposer = factory.get().createRecomposer(rootView)
  			//将其赋值给rootView的compositionContext，而compositionContext也是一个扩展函数，通用使用tag保存
        rootView.compositionContext = newRecomposer
  		...
}
```

> #### View.compositionContext
>
> ```kotlin
> var View.compositionContext: CompositionContext?
>     get() = getTag(R.id.androidx_compose_ui_view_composition_context) as? CompositionContext
>     set(value) {
>         setTag(R.id.androidx_compose_ui_view_composition_context, value)
>     }
> ```

在这里，我们知道了` resolveParentCompositionContext()` 实际上是初始化了ComposeView的扩展变量compositionContext。并且我们得到了这个返回值 `parentContext `。 



#### VewGroup.setContent()

接着我们继续回到setContent中，去看看：

> **👉2. ** `setContent(resolveParentCompositionContext())`
>
> ```kotlin
> internal fun ViewGroup.setContent(
>     parent: CompositionContext,
>     content: @Composable () -> Unit
> ): Composition {
>     GlobalSnapshotManager.ensureStarted()
>     val composeView =
>         if (childCount > 0) {
>           	//第一次调用这里肯定为null，注意下面方法调用
>             getChildAt(0) as? AndroidComposeView
>         } else {
>             removeAllViews(); null
>           //初始化一个AndroidComposeView，并将我们最外面的content函数传入，
>             //将初始化的AndroidComposeView添加到ComposeView上
>         } ?: AndroidComposeView(context).also { addView(it.view, DefaultLayoutParams) }
>   	//这里留一个伏笔，即我们的下个阶段将
>     return doSetContent(composeView, parent, content)
> }
> ```
>
> 如上所述，这里拿到当前viewGroup即 ComposeView ,然后判断childCount ，即是否有子view,因为是第一次，所以肯定执行到了 flase.  
>
> 即执行到了 `AndroidComposeView(context).also { addView(it.view, DefaultLayoutParams) }` ，也就是生成了一个AndroidComposeView 的ViewGroup容器，内部构造函数中的context，正是我们第三部的content(),也就是我们自己的业务代码。然后调用ComposeView的addView方法，将自己添加到ComposeView中。
>
> 到这里为止，如果你还记得我们最开始的布局层级，那就应该能明白最基础的流程。
>
> <img src="https://tva1.sinaimg.cn/large/008i3skNly1gs3ynr82wrj30vg0g6tce.jpg" alt="image-20210703185751007" style="zoom:50%;" />

### 小总结

当我们调用Compsoe的setContent 之后，其内部先判断当前的基础(R.id.content)的View是不是ComposeView,如果不是则初始化一个，并且调用其的setContent方法，将 shouldCreateCompositionOnAttachedToWindow 置为true,并且将最开始传入的content函数赋值给Compose内部的变量content，接着使用Activity的setContentView，将初始化的ComposeView添加到底层布局R.id.content上；在view完全可见时，即onAttachView被调用时，开始去初始化当前compose页面，其内部初始化了一个名为AndroidComposeView的子View,然后调用我们传入的content函数，生成一个content，将其作为构造函数传入AndroidComposeView中，从而生成了子view。然后将其add到了ComposeView上。从而完成了布局的初始化。





## 探索 - doSetContent

```kotlin
private fun doSetContent(
  	//ComposeView->child
    owner: AndroidComposeView,
  	//最顶层ComposeView 的扩展属性 
    parent: CompositionContext,
    content: @Composable () -> Unit
): Composition {
    ...
  	//关注点 1 初始化Composition
    val original = Composition(UiApplier(owner.root), parent)
  	//初始化AndroidComposeView的tag
    val wrapped = owner.view.getTag(R.id.wrapped_composition_tag)
        as? WrappedComposition
  			// 关注点 2
  			// 注意这里传入的 original ，其实例是 CompositionImpl
        ?: WrappedComposition(owner, original).also {
            owner.view.setTag(R.id.wrapped_composition_tag, it)
        }
  	//关注点 3. 
    wrapped.setContent(content)
    return wrapped
}

```

#### 

### 1. Composition(UiApplier(owner.root), parent)

  注意这里初始化Composition时传递进了parent,而parent在最上面的 `createAndInstallWindowRecomposer()` 方法里
  所以parent其实是Recomposer对象。

```kotlin
// -> 最开始初始化的地方
internal fun createAndInstallWindowRecomposer(rootView: View): Recomposer {
  			//使用默认的工厂创建一个Recomposer
        val newRecomposer = factory.get().createRecomposer(rootView)
  			//将其赋值给rootView的compositionContext，而compositionContext也是一个扩展函数，通用使用tag保存
        rootView.compositionContext = newRecomposer
  		...
}

//关注点 -> Composition()
fun Composition(
    applier: Applier<*>,
    parent: CompositionContext
): Composition =
    CompositionImpl(parent,applier)

//关注点 -> CompositionImpl
internal class CompositionImpl(
    private val parent: CompositionContext
		...
) 
```



### 2.  WrappedComposition

```kotlin
 override fun setContent(content: @Composable () -> Unit) {
        owner.setOnViewTreeOwnersAvailable {
              			...
          					//主要关注这里 original -> 
                    original.setContent {
                    }
    }
 }
```

##### CompositionImpl-setContent

```kotlin
override fun setContent(content: @Composable () -> Unit) {
    check(!disposed) { "The composition is disposed" }
  	//将我们最开始传入的content函数传给 composable
    this.composable = content
  	//关注点在这里,parent是CompositionContext对象，实际是 Recomposer的实例
    parent.composeInitial(this, composable)
}

private val parent: CompositionContext
```



### Recomposer-composeInitial()

```kotlin
internal override fun composeInitial(
    composition: ControlledComposition,
    content: @Composable () -> Unit
) {
    val composerWasComposing = composition.isComposing
  	//关注点 1. 
    composing(composition, null) {
      	//关注点2. 
        composition.composeContent(content)
    }
   	...
}
```

#### 2. composition.composeContent()

```kotlin
override fun composeContent(content: @Composable () -> Unit) {
    synchronized(lock) {
        drainPendingModificationsForCompositionLocked()
        composer.composeContent(takeInvalidations(), content)
    }
}
```

##### ComposerImpl-composeContent()

```kotlin
internal fun composeContent(
    invalidationsRequested: IdentityArrayMap<RecomposeScopeImpl, IdentityArraySet<Any>?>,
    content: @Composable () -> Unit
) {
    doCompose(invalidationsRequested, content)
}
```

##### doCompost()

```kotlin
private fun doCompose(
    invalidationsRequested: IdentityArrayMap<RecomposeScopeImpl, IdentityArraySet<Any>?>,
    content: (@Composable () -> Unit)?
) {
    		...
  					//从根开始重组
            startRoot()
            // Ignore reads of derivedStatOf recalculations
            observeDerivedStateRecalculations(
                ...
            ) {
                if (content != null) {
                  	// 开始组装
                    startGroup(invocationKey, invocation)
                  	//关注点
                    invokeComposable(this, content)
                  	//结束组装
                    endGroup()
                } else {
                  	//跳过当前组装
                    skipCurrentGroup()
                }
            }
            endRoot()
            complete = true
}
```

##### invokeComposable()

```kotlin
internal fun invokeComposable(composer: Composer, composable: @Composable () -> Unit) {
    @Suppress("UNCHECKED_CAST")
  	//编译器会在编译后插入一些其他参数，所以可以强制为Function2,以便直接调用
    val realFn = composable as Function2<Composer, Int, Unit>
  	//执行我们最开始传入的content,即我们自己的组件开始被重组
    realFn(composer, 1)
}
```

### 小结

当执行到doSetContent时,其最终是为了让 ComposerImpl 调用 doCompose(),从而调用invokeComposable()方法，并传入了content()函数，并最终在里面调用，完成组件的构建。



我们从上面开始一步步看到了整个compose运行流程，那么最终的问题来了，我们自己写的Text() 这些究竟是怎样被解析的呢?

## 解惑 - 我们的Compose如何被安装？

我们通过最开始的示例Box,向上找，找最顶层的View,即 Layout .

### Layout

```kotlin
@Suppress("ComposableLambdaParameterPosition")
@Composable inline fun Layout(
  	//我们自己写的业务组件
    content: @Composable () -> Unit,
  	//修饰符
    modifier: Modifier = Modifier,
  	//测量规则
    measurePolicy: MeasurePolicy
) {
    val density = LocalDensity.current
    val layoutDirection = LocalLayoutDirection.current
  	//关注点1
    ReusableComposeNode<ComposeUiNode, Applier<Any>>(
      	//关注点2 . 
        factory = ComposeUiNode.Constructor,
        update = {
            set(measurePolicy, ComposeUiNode.SetMeasurePolicy)
            set(density, ComposeUiNode.SetDensity)
            set(layoutDirection, ComposeUiNode.SetLayoutDirection)
        },
        skippableUpdate = materializerOf(modifier),
        content = content
    )
}
```

#### 关注点1

**factory = ComposeUiNode.Constructor**

```kotlin
val Constructor: () -> ComposeUiNode = LayoutNode.Constructor

internal val Constructor: () -> LayoutNode = { LayoutNode() }
```

我们发现，factory工厂实例默认就是 ComposeUiNode 的静态函数变量，其函数内部是用于初始化一个 `LayoutNode`



### ReusableComposeNode

```kotlin
@Composable @ExplicitGroupsComposable
inline fun <T, reified E : Applier<*>> ReusableComposeNode(
    noinline factory: () -> T,
    update: @DisallowComposableCalls Updater<T>.() -> Unit,
    noinline skippableUpdate: @Composable SkippableUpdater<T>.() -> Unit,
    content: @Composable () -> Unit
) {
    if (currentComposer.applier !is E) invalidApplier()
    currentComposer.startReusableNode()
    if (currentComposer.inserting) {
      	//创建node，并将其插入到rootNode里面去，具体参考下面分析
        currentComposer.createNode(factory)
    } else {
        currentComposer.useNode()
    }
    currentComposer.disableReusing()
    Updater<T>(currentComposer).update()
    currentComposer.enableReusing()
    SkippableUpdater<T>(currentComposer).skippableUpdate()
    currentComposer.startReplaceableGroup(0x7ab4aae9)
    content()
    currentComposer.endReplaceableGroup()
    currentComposer.endNode()
}
```



#### createNode()

```kotlin
@Suppress("UNUSED")
override fun <T> createNode(factory: () -> T) {
    ...
    recordFixup { applier, slots, _ ->
        @Suppress("UNCHECKED_CAST")
        //关注点1 调用传入factory(),获得LayoutNode
        val node = factory()
        slots.updateNode(groupAnchor, node)
        //关注点2
        //这里的applier 就是 dosetContent中插入的- 
                 //    val original = Composition(UiApplier(owner.root), parent)
                 // 实际上 nodeApplier就是 UiApplier
        @Suppress("UNCHECKED_CAST") val nodeApplier = applier as Applier<T>
        //伪插入代码
        nodeApplier.insertTopDown(insertIndex, node)
        applier.down(node)
    }
    recordInsertUpFixup { applier, slots, _ ->
        @Suppress("UNCHECKED_CAST")
        val nodeToInsert = slots.node(groupAnchor)
        applier.up()
        @Suppress("UNCHECKED_CAST") val nodeApplier = applier as Applier<Any?>
        //插入到当前子LayoutNode
        nodeApplier.insertBottomUp(insertIndex, nodeToInsert)
    }
}
```

#### 关注点1

```kotlin
internal class UiApplier(
    root: LayoutNode
) : AbstractApplier<LayoutNode>(root)
```

我们在dosetContent里面,有下面这样一行代码：

`val original = Composition(UiApplier(owner.root), parent)`

而这里是生成了AndrooidComposeView的 Composition。

而owner.root又是一个LayoutNode，如下所示：

```kotlin
override val root = LayoutNode().also {
    it.measurePolicy = RootMeasurePolicy
    it.modifier = Modifier
        .then(semanticsModifier)
        .then(_focusManager.modifier)
        .then(keyInputModifier)
}
```

再结合我们关注点1，拿到的node, 综合下来其实就是，把我们新生成的node插入到了rootNode里，即插入到了AndroidComposeView对应的 LayoutNode 里。





## 归一反思

整个流程相当而言，错综复杂，总结下来我们可以理解为，Compose的整个流程如下：

```
DecorView
	LinearLayout
		R.id.content
						👇
			ComposeView
				AndroidComposeView -> 
								👇
					 root:LayoutNode
								 LayoutNode [
								  		Text()
								  		Image()
								 ]
```

​    



**Compose**-> 我们的代码流程

- SlotTable 存的是全量信息
- Snapshot 现在状态与下一个状态信息
- 调用处 Text("123")
- SlotTable
- 显示处 LayoutNode



## @Composeable 使用规则

- 所有加了 @Composable 注解的函数，都需要在别的加了 @Composable 的函数里面才能调用；
- 所有内部没调用任何其他 @Composable 函数的函数，都没必要加上这个注解，因为没意义。

什么是Composable?

带了 @Composable 注解的方法就叫Composable控件。类似kotlin-协程中的挂起函数。
