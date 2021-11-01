# 浅析Compose是如何安装到传统Android视图上

## Hi , :)

看完本文可以帮你解开什么问题？

- 为什么 `Compose` 无需在意 `view` 层级问题，怎样嵌套都行？ (最简单10s就能明白);
- `Compose` 如何安装到传统 `View` 视图上;

## 门外汉-从布局窥一眼

这是一段 `Compose` 的简单代码，我们演示了多层嵌套下的示例:

![image-20210703184658154](https://tva1.sinaimg.cn/large/008i3skNly1gu5wxsxvinj61ke0u0n2l02.jpg)

如果按照传统 `View` 的思维，我们不难发现，当前 **content(R.id.content(FrameLayout)->)** 布局中存在5层嵌套,这是极不可取的一种做法。

但是现在是 `Compose` ,最终的绘制真的会有5层吗？

我们打开 **Filpper** 看一下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gu5wxt9nm0j60vg0g6gnd02.jpg" alt="image-20210703185751007" style="zoom:50%;" />

显然 **R.id.content** 下只有一个 `ComposeView` ，然后内部包含了一个 `AndroidComposeView` ，我们上述中的 `Box` 最终都被解析并安装到了这个自定义view上。

所以我们简单点可以总结为：

**JetPack-Compose** 其自定义了一个 基础容器- `ComposeView` ，以及其他扩展View，比如 `AndroidComposeView` ,并对其进行封装，对外提供了各种我们在上层所使用的各种组件或者容器。

所以当我们在 `Compose` 中 `setContent` 后，其初始化了一个 `ComposeView` ，并且添加了一个 `AndroidComposeView` ,其承载了我们代码中所写的全部组件，并进行解析，最终绘制在了传统UI中。

**所以为什么说Compose不在意布局层级呢？**

> 因为人家只有两层啊，即业务代码中，`ComposeView` 下就只有一个 `AndroidComposeView` ,而其他 `Image`,`Box` 等组件都是人家自己绘制的。你说相比 **传统View** 还会存在层级问题吗😂

**一些猜测：**

为什么叫 `AndroidComposeView` 呢？

> `Compose` 现在不仅仅支持 `Android`，现在预览版也支持 `Desktop` ,所以很可能 `ComposeView` 很可能还会涉及其他平台系统。所以最顶层是 `ComposeView` ,而对于 `Android` 的支持为 `AndroidComposeView` 。
>
> 当然上述只是我一个猜测，如果大佬们有其他想法，也欢迎分享。

## 解析-setContent内部实现

我们在上面知道了 `Compose` 最终在 `Android View` 的展现形式，那么它到底是怎样设置上去的呢，接下来我们就简单解析一下，不涉及`Compose` 相关过多源码，比较好理解：

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

- **setParentCompositionContext()**
- **setContent()**

---

### ComposeView

#### setParentCompositionContext

设置视图合成的父级 `context` ，里面仅仅是一个赋值，暂时跳过

```kotlin
fun setParentCompositionContext(parent: CompositionContext?) {
        parentContext = parent
}
```

---

#### setContent

设置 `compose UI ` 内容，当视图被添加到窗口时调用。

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

我们接下来去看看 `AbstractComposeView` 的 `onAttachToWindows()` 方法。

---

### AbstractComposeView 

#### onAttachToWindows

当被安装到windows上时调用。

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

##### ensureCompositionCreated

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

---

#### resolveParentxxxContext

其作用是解析父组合上下文。

```kotlin
private fun resolveParentCompositionContext() = parentContext
		//👉1. 
    ?: findViewTreeCompositionContext()?.also { cachedViewTreeCompositionContext = it }
    ?: cachedViewTreeCompositionContext
		// 👉2. 
    ?: windowRecomposer.also { cachedViewTreeCompositionContext = it }
```

##### 1. findViewTreexxxContext

查找当前view树的父context

```kotlin
fun View.findViewTreeCompositionContext(): CompositionContext? { 
	//先取自己的compositionContenxt，如果取到直接返回，否则不断向上找父级的context.
var found: CompositionContext? = compositionContext
if (found != null) return found
var parent: ViewParent? = parent
while (found == null && parent is View) {
  found = parent.compositionContext
  parent = parent.getParent()
}
return found
}
```

###### View.compositionContext

`compositionContext` 是一个扩展函数，内部使用tag保存当前context

var View.compositionContext: CompositionContext?
get() = getTag(R.id.androidx_compose_ui_view_composition_context) as? CompositionContext
set(value) {
  setTag(R.id.androidx_compose_ui_view_composition_context, value)
}

---

##### 2. windowRecomposer

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

###### View.contentChild 

```kotlin
private val View.contentChild: View
get() {
  var self: View = this
  var parent: ViewParent? = self.parent
  while (parent is View) {
      if (parent.id == android.R.id.content) return self
      self = parent
      parent = self.parent
  }
  return self
}
```

###### createAndxxxRecomposer(rootView)

创建一个 `Recomposer`，并且将其赋值给rootView的扩展变量 `compositionContext`

```kotlin
internal fun createAndInstallWindowRecomposer(rootView: View): Recomposer {
  			//使用默认的工厂创建一个Recomposer
        val newRecomposer = factory.get().createRecomposer(rootView)
  			//将其赋值给rootView的compositionContext，而compositionContext也是一个扩展函数，通用使用tag保存
        rootView.compositionContext = newRecomposer
  		...
}

-- View.compositionContext
var View.compositionContext: CompositionContext?
get() = getTag(R.id.androidx_compose_ui_view_composition_context) as? CompositionContext
set(value) {
  setTag(R.id.androidx_compose_ui_view_composition_context, value)
}
```

在这里，我们知道了` resolveParentCompositionContext()` 实际上是初始化了 `ComposeView` 的扩展变量 `compositionContext` 。并且我们得到了这个返回值 `parentContext `。 

---

### ViewGroup.setContent

接着我们继续回到setContent中，去看看：

> **👉3.** `setContent(resolveParentCompositionContext())`

```kotlin
internal fun ViewGroup.setContent(
parent: CompositionContext,
content: @Composable () -> Unit
): Composition {
GlobalSnapshotManager.ensureStarted()
val composeView =
  if (childCount > 0) {
    	//第一次调用这里肯定为null，注意下面方法调用
      getChildAt(0) as? AndroidComposeView
  } else {
      removeAllViews(); null
    //初始化一个AndroidComposeView，并将我们最外面的content函数传入，
      //将初始化的AndroidComposeView添加到ComposeView上
  } ?: AndroidComposeView(context).also { addView(it.view, DefaultLayoutParams) }
	//这里留一个伏笔，即我们的下个阶段将
return doSetContent(composeView, parent, content)
}
```

如上所述，这里拿到当前 `viewGroup` 即 `ComposeView` ,然后判断 `childCount` 否有子view,因为是第一次，所以肯定执行到了 `flase`.  

即执行到了 `AndroidComposeView(context).also { addView(it.view, DefaultLayoutParams) }` ，也就是生成了一个`AndroidComposeView` 的 `ViewGroup` 容器，内部构造函数中的`context` 正是我们第三步的 **content()** ,也就是我们自己的业务代码。然后调用 `ComposeView` 的 `addView()` 方法，将自己添加到 `ComposeView` 中。

到这里为止，如果你还记得我们最开始的布局层级，那就应该能明白最基础的流程。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gu5wxt9nm0j60vg0g6gnd02.jpg" alt="image-20210703185751007" style="zoom:50%;" />

### 总结

1. 当我们调用 `Compsoe` 的 `setContent()` 之后，其内部先判断当前的基础 **(R.id.content)** 的 `View` 是不是 `ComposeView` ，如果不是则初始化一个，并且调用其的 `setContent()` 方法，将 `shouldCreateCompositionOnAttachedToWindow` 置为 `true` ,并且将最开始传入的 `content` 函数赋值给 `Compose` 内部的变量 `content` 。
2. 接着使用 `Activity` 的 `setContentView()` ，将初始化的 `ComposeView` 添加到底层布局 **R.id.content** 上；
3. 在 `view` 完全可见时，即 `onAttachView` 被调用时，开始去初始化当前 `compose` 页面，其内部初始化了一个名为 `AndroidComposeView` 的子View。然后调用我们传入的 `content()` 函数，生成一个 `content` ，将其作为构造函数传入 `AndroidComposeView` 中，从而生成了子view。然后将其 **add** 到了 `ComposeView` 上。从而完成了布局的初始化。

## 碎碎念

## 

本文是理解 `Compose` 设计中比较简单的一篇，适合初学的同学简单了解 **Compose与View** 的相爱相杀。后续我将继续深追 `Compose` 的部分源码设计以及在实际落地中的场景解决方案。

如果本文对你有所帮助，欢迎点赞支持一下，大家加油 :)

