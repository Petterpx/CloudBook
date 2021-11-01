# JetPack Compose主题配色太少怎么办，来设计自己的颜色系统吧



## 引言

`JetPack Compose` 正式版已经发布好几个月了，在这段时间里，除了业务相关需求之外，我也开始了 `Compose` 在实际项目中的落地实验，因为一旦要接入当前项目，那么遇到的问题其实远远大于新创建一个项目所需要的问题。

本篇要解决的就是 `Compose` 默认 `Material` 主题颜色太少，**如何配置自己的业务颜色板**，或者说，如何自定义自己的颜色系统，并由点入深，系统的分析相关实现方法与原理。

## 问题

在开始之前，我们先看看目前创建一个 `Compose` 项目，默认的 `Material` 主题为我们提供的颜色有哪些:

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrpq3tgdwj30nu0m2tad.jpg" width = "400" height = "300" alt="图片名称" align=center />



对于一个普通的应用而言，默认的已基本满足开发使用，基本的主题配色已经足够。但是此时一个问题出现了，如果我存在其他的主题配色呢?

### 传统做法

在传统的 `View` 体系中，我们一般都会将颜色定义在 **color.xml** 文件中，在使用的时候直接读取即可，**getColor(R.xx)** ,这个大家都已经很熟悉了，那么在 `Compose` 中呢？

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrks7ahlxj61500ey0wc02.jpg" alt="image-20211025151217426" style="zoom:50%;" />

### Compose

在 `Compose` 中，`google` 将颜色数值统一放在了 `theme` 下的 **color.kt** 中，这其实也就是全局静态变量，乍一看好像没什么问题，那我的业务颜色放在那里呢，总不能都全局暴露吧？

![image-20211025151546986](https://tva1.sinaimg.cn/large/008i3skNly1gvrkvuempij61yo0h841902.jpg)

但是聪明的你肯定知道，我按照老办法放到 **color.xml** 里不就行哈，这样也不是不可以，但是随之而来的问题如下：

- 切换主题时候，颜色怎么统一解决？
- 在 `Google` 的 **simple** 里，**color.xml** 里往往不会写任何配置，即 Google 本身不建议在 `compose` 里这样用

那么我该怎么办，我去看看google的simple,看看他们怎么解决：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrl66n3y4j618g0rktbt02.jpg" alt="image-20211025152543420" style="zoom:50%;" />

simple果然是simple 😑 ,`Google` 完全按照 `Material` 的标准，即不会出现其他的非主题配色，那实际呢，我们开发怎么办。然后我搜了下目前github上大佬们写的一些开源项目，发现他们也都是按照 `Material` 去实现，但是很明显这样很不符合实际(国情)。🙃



## 解决思路

### 随心所欲写法(不推荐)

> 形容 没什么标准，直接卷起袖子撸代码，左脑思考，右手开敲，拿起 ⌨️ 就是干，又指新时代埋头苦干的 👷🏻‍♂️

既然官方没写怎么解决，那就自己想办法解决喽。

`compose` 中，对于数据的改变监听是使用 `MutableState` ，那么我自己自定义一个单例持有类，持有现有的主题配置，然后定义一个业务颜色类，并且定义相应的主题颜色类对象，最终根据当前单例的主题配置，去判断最终使用什么颜色板即可，更改业务主题时只需要更改这个单例主题配置字段即可。一想到如此简单，我可真是个抖机灵，说干就干 👨‍🔧‍

##### 创建主题枚举

```kotlin
enum class ColorPallet {
  	// 默认就给两个颜色，根据需求可以定义多个
    DARK, LIGHT
}
```

##### 增加主题配置单例

```kotlin
object ColorsManager {
    /** 使用一个变量维护当前主题 */
    var pallet by mutableStateOf(ColorPallet.LIGHT)
}
```

##### 增加颜色板

```kotlin
/** 共用的颜色 */
val Purple200 = Color(0xFFBB86FC)
val Purple500 = Color(0xFF6200EE)
val Purple700 = Color(0xFF3700B3)
val Teal200 = Color(0xFF03DAC5)

/** 业务颜色配置,如果需要增加其他主题业务色,直接定义相应的下述对象即可,如果某个颜色共用,则增加默认值 */
open class CkColor(val homeBackColor: Color, val homeTitleTvColor: Color = Color.Gray)

/** 提前定义好的业务颜色模板对象 */
private val CkDarkColor = CkColor(
    homeBackColor = Color.Black
)

private val CkLightColor = CkColor(
    homeBackColor = Color.White
)

/** 默认的颜色配置,即Md的默认配置颜色 */
private val DarkColorPalette = darkColors(
    primary = Purple200,
    primaryVariant = Purple700,
    secondary = Teal200
)
private val LightColorPalette = lightColors(
    primary = Purple500,
    primaryVariant = Purple700,
    secondary = Teal200
)
```

##### 增加统一调用入口

为了便于实际使用，我们增加一个 MaterialTheme.ckColor的扩展函数，以便使用我们自定义的颜色组：

```kotlin
/** 增加扩展 */
val MaterialTheme.ckColor: CkColor
    get() = when (ColorsManager.pallet) {
        ColorPallet.DARK -> CkDarkColor
        ColorPallet.LIGHT -> CkLightColor
    }
```

##### 最终的主题如下

```kotlin
@Composable
fun CkTheme(
    pallet: ColorPallet = ColorsManager.pallet,
    content: @Composable() () -> Unit
) {
    val colors = when (pallet) {
        ColorPallet.DARK -> DarkColorPalette
        ColorPallet.LIGHT -> LightColorPalette
    }
    MaterialTheme(
        colors = colors,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

#### 效果图

 <center class="half">  <img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrpbq6i2wj30w20u0q5s.jpg" width="400"/>  <img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrpf53z1zg30aa0je19q.gif" width="200"/>  </center>



看效果也还成，简单粗暴，[看着] 也没什么问题，那有没有什么其他方式呢？我还是不相信官方没有写，可能是我疏忽了。

### 自定义颜色系统(官方)

就在我翻官方文档时，突然看见了这样几个小字，它实现了[自定义颜色系统](https://github.com/android/compose-samples/tree/main/Jetsnack)。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvrpsxzqm1j31g20qeq6m.jpg" alt="image-20211025180600191" style="zoom: 33%;" />

真是瞎了自己的眼，居然没看到这行字，有了官方示例，于是就赶紧去学习(抄)代码。

##### 增加颜色模板

```kotlin
// 示例，正确做法是放到color.kt下
val Blue50 = Color(0xFFE3F2FD)
val Blue200 = Color(0xFF90CAF9)
val A900 = Color(0xFF0D47A1)

/**
 * 实际主题的颜色集,所有颜色都需要添加到其中,并使用相应的子类覆盖颜色。
 * 每一次的更改都需要将颜色配置在下方 [CkColors] 中,并同步 [CkDarkColor] 与 [CkLightColor]
 * */
@Stable
class CkColors(
    homeBackColor: Color,
    homeTitleTvColor: Color
) {
    var homeBackColor by mutableStateOf(homeBackColor)
        private set
    var homeTitleTvColor by mutableStateOf(homeTitleTvColor)
        private set

    fun update(colors: CkColors) {
        this.homeBackColor = colors.homeBackColor
        this.homeTitleTvColor = colors.homeTitleTvColor
    }

    fun copy() = CkColors(homeBackColor, homeTitleTvColor)
}

/** 提前定义好的颜色模板对象 */
private val CkDarkColors = CkColors(
    homeBackColor = A900,
    homeTitleTvColor = Blue50,
)

private val CkLightColors = CkColors(
    homeBackColor = Blue200,
    homeTitleTvColor = Color.White,
)
```

##### 增加 xxLocalProvider

```kotlin
@Composable
fun ProvideLcColors(colors: CkColors, content: @Composable () -> Unit) {
    val colorPalette = remember {
        colors.copy()
    }
    colorPalette.update(colors)
    CompositionLocalProvider(LocalLcColors provides colorPalette, content = content)
}
```

##### 增加 LocalLcColors 静态变量

```kotlin
// 创建静态 CompositionLocal ,通常情况下主题改动不会很频繁
private val LocalLcColors = staticCompositionLocalOf {
    CkLightColors
}
```

##### 增加主题配置单例

```kotlin
enum class StylePallet {
  	// 默认就给两个颜色，根据需求可以定义多个
    DARK, LIGHT
}

/* 针对当前主题配置颜色板扩展属性 */
private val StylePallet.colors: Pair<Colors, CkColors>
    get() = when (this) {
        StylePallet.DARK -> DarkColorPalette to CkDarkColors
        StylePallet.LIGHT -> LightColorPalette to CkLightColors
    }


/** CkX-Compose主题管理者 */
object CkXTheme {
    /** 从CompositionLocal中取出相应的Local */
    val colors: CkColors
        @Composable
        get() = LocalLcColors.current
  
  	/** 使用一个state维护当前主题配置,这里的写法取决于具体业务，如果你使用了深色模式默认配置，则无需这个变量，即app只支持深色与亮色，那么只需要每次读系统配置即可。但是compose本身可以做到快速切换主题，那么维护一个变量是肯定没法避免的 */
    var pallet by mutableStateOf(StylePallet.LIGHT)
}
```

##### 最终主题代码

```kotlin
@Composable
fun CkXTheme(
    pallet: StylePallet = CkXTheme.pallet,
    content: @Composable () -> Unit
) {
    val (colorPalette, lcColors) = pallet.colors
    ProvideLcColors(colors = lcColors) {
        MaterialTheme(
            colors = colorPalette,
            typography = Typography,
            shapes = Shapes,
            content = content
        )
    }
}
```

#### 分析

最终的效果和上述的一致，也就不具体赘述了，我们主要来分析一下，为什么Google要这么写：

我们可以看到上述的示例里主要是使用了 `CompositionLocalProvider` 去保存当前的主题配置 ,而 `CompositionLocalProvider` 又继承自 `CompositionLocal` ，比如我们常用的 `MaterialTheme` 主题中的 `Shapes` ，`typography` 都是由此来管理。

`CkColors` 这个类上增加了 **@Stable** ,其代表着对于 `Compose` 而言，这个类是一个稳定的类，即每次更改不会引发重组，内部的颜色字段使用了 `mustbaleStateOf` 包装，以便当颜色更改时触发重组，内部还增加了 **update()** 与 **copy()** 方法，以便于管理与单向数据的更改。

其实如果我们去看的 `Colors` 类。就会发现上述示例中的 `CkColors` 和其是完全一样的设计方式。

![image-20211027155601502](https://tva1.sinaimg.cn/large/008i3skNly1gvtxadje6tj32eu0mcn1c.jpg)

> 所以在Compose中自定义主题颜色，其实就是我们在 `Colors` 的基础上自己又写了一套自己的配色。😂

**既然这样，那为什么我们不直接继承Colors去增加配色呢？使用的时候我强制一下不就行,这样不就不用再自己造什么 CompositionLocal 了？** 

其实很好理解，因为 `Colors` 中的 **copy()** 以及 **update()** 无法被重写，即没加 `open` ，而且其内部变量使用了 `internal` 修饰 `set` 方法。更重要的原因是这样 **不符合Md的设计** ，所以这也就是为什么 需要我们去自定义自己的颜色系统，甚至于可以完全自定义自己的主题系统。前提是你觉得自定义的主题里面再包装一层 `MaterialTheme` 主题比较丑陋的话,当然相应的，你也需要考虑如何解决其他附带的问题。

## 反思

我们上面说的都是使用方面的，那么你有没有想过？为什么官方自定义设计系统要使用 `CompositionLocal` 呢？

可能有新同学并没有使用过这个，为了便于更好理解，首先我们先搞清楚 `CompositionLocal` 是干什么的，先不说其通俗概念，我们简单用个小例子就能讲明白。

### 解构

在常见的开发场景中，我们很多时候，经常会将某个参数传递给其他方法，我们称之为显示传递。

切换一下场景，我们在 `Compose` 中，经常会给可组合函数传递参数，于是这个方式被 Google 学术化称为: **数据以参数的形式 向下流经 整个界面树传递给每个可组合函数** ，就如下述所示：

```kotlin
@Composable
fun A(message: String) {
  Column() {
     B(message)
  }
}

@Composable
fun B(message: String) {
    Text(text = message)
    C(message)
}

@Composable
fun C(message: String) {
    Text(text = message)
}
```

上述示例中有3个可组合函数，其中 **A** 需要接收一个 `message` 字符串，并且将其传递给 **B** ，而 **B** 同时又需要传递给 C ，类似于无限套娃一样，此时我们可能感觉还行吧，但是如果这种套娃出现 n 层呢，但是数据如果不止一个呢？此时将可能非常麻烦。

那么有没有其他方式解决呢？在 `Compose` 中，官方给出了标准答案，那就是 `CompositionLocal` ：

即使用 `CompositionLocal` 来完成 `composable` 树中的数据共享，并且 `CompositionLocal` 具有层级，它可以被限定在某个 `composable` 作为根节点的子树中，默认向下传递，同时子树中的某个 `composable` 也可以对该 `CompositionLocal` 进行覆盖，然后这个新值就会在这个 `composable` 中继续向下传递。

> `composable` 即可组合函数，简单理解就是使用了 `@Composable` 标注的方法。

### 实践

如下所示，我们对上述代码使用 `CompositionLocal` 进行改造：

```kotlin
val MessageContent = compositionLocalOf { "simple" }

@Composable
fun A(message: String) {
    Column() {
        // provides 相当于写入数据
        CompositionLocalProvider(MessageContent provides message) {
            B()
        }
    }
}

@Composable
fun B() {
    Text(text = MessageContent.current)
    CompositionLocalProvider(MessageContent provides "临时对值进行更改") {
        C()
    }
}

@Composable
fun C() {
    Text(text = MessageContent.current)
}
```

先定义了一个名为 `MessageContent` 的 `CompositionLocal` ，其默认值为 **"simple"** , **A** 方法接收一个 message 字段，并且将其写入 `MessageContent` ,然后在 **B** 中，我们就可以获取到刚才方法 **A** 中写入到CompositionLocal的数据，而无需显示的在方法参数里增加字段。

同样，方法B也可以对这个 `CompositionLocal` 进行更改，这样 **C** 就会拿到另一个值。

并且当我们使用 `CompositionLocal.current` 来获取数据的时候，这个 `current` 会返回距离当前组件最近的那一个值，所以我们也可以利用其来做一些隐式分离的基础实现。

### 扩展

相应的,我们再说一下创建 `CompositionLocal` 的方式：

- [`compositionLocalOf`](https://developer.android.google.cn/reference/kotlin/androidx/compose/runtime/package-summary#compositionLocalOf(androidx.compose.runtime.SnapshotMutationPolicy,kotlin.Function0))：在重组期间更改提供的值只会使读取其 [`current`](https://developer.android.google.cn/reference/kotlin/androidx/compose/runtime/CompositionLocal#current()) 值的内容无效。
- [`staticCompositionLocalOf`](https://developer.android.google.cn/reference/kotlin/androidx/compose/runtime/package-summary#staticCompositionLocalOf(kotlin.Function0))：与 `compositionLocalOf` 不同，`Compose` 不会跟踪 `staticCompositionLocalOf` 的读取。更改该值会导致提供 `CompositionLocal` 的整个 `contentlambda` 被重组，而不仅仅是在组合中读取 `current` 值的位置。

### 总结

我们在上面大概了解了 `CompositionLocal` 的作用，试想一下，如果不用它，如果让我们自己去实现一个颜色系统，可能就会陷入我们最开始那种 随心所欲 的写法。

首先，那种写法可以用吗？当然可以用，但是实际中问题会很多，比如说主题的更改会导致而且不符合 `Compose` 的设计，而且如果我们可能有一部分业务在一定情况下，它可能一直保持一个主题色，那么此时怎么解决？

> 如果是方法1，可能此时会进入硬编码阶段，即使用复杂的业务逻辑去完成; 但如果是利用 `CompositionLocal` 呢？这个问题还会存在吗，只需要写入一个新的颜色配置即可，这个逻辑结束后再重新写入当前主题配置即可，还会存在复杂的逻辑缠绕情况吗？
>
> 这也就是为什么 `Google` 选择使用 `CompositionLocal` 去自定义颜色系统以及整个主题系统中可以供用户操纵的配置，即隐式，对使用者而言，无感知的就可以办到。

## 深入分析

看完了 `CompositionLocal` 的妙用与实际场景，我们不妨想一想，`CompositionLocal` 到底是怎么实现的，所谓知其然知其所以然。不深入源码往往很难理解具体实现，所以这部分的解析可能略显复杂。大家如果觉得晦涩，不妨先看一下 Android开发者-[深入详解Jetpack Compose实现原理](https://mp.weixin.qq.com/s/CENFvRrZy6Tzhr3-FZIeNg)，再来理解下面的某些术语，可能会更简单点，因本篇不是通俗的讲 `compose` 实现原理，所以大家参阅上面的链接即可。

### CompositionLocal

言归正传，我们先看一眼源码，相应的注释与代码如下：

```kotlin
sealed class CompositionLocal<T> constructor(defaultFactory: () -> T) {
   
  	// 默认值
    internal val defaultValueHolder = LazyValueHolder(defaultFactory)
  	
  	// 写入最新数据
    @Composable
    internal abstract fun provided(value: T): State<T>
		
  	// 返回由最近的 CompositionLocalProvider 提供的值
    @OptIn(InternalComposeApi::class)
    inline val current: T
        @ReadOnlyComposable
        @Composable
  			//直接追这里代码
        get() = currentComposer.consume(this)
}
```

我们知道获取数据的时候使用的 `current` ,那么直接追这里即可。

#### currentComposer.consume()

```kotlin
@InternalComposeApi
override fun <T> consume(key: CompositionLocal<T>): T =
		// currentCompositionLocalScope() 获取父可组合项提供的当前CompositionLocal范围map
    resolveCompositionLocal(key, currentCompositionLocalScope())
```

这段代码主要用于解析本地可组合项，从而获得数据。我们先看里面的 **currentCompositionLocalScope()**

#### currentCompositionLocalScope()

```kotlin
private fun currentCompositionLocalScope(): CompositionLocalMap {
  	//如果可组合项当前正在插入并且有数据提供者
    if (inserting && hasProvider) {
      	// 从插入表中取离当前 composable 最近的group（可以理解为直接去取index最近的currentGroup）
        var current = writer.parent
      	// 如果这个group不是空的
        while (current > 0) {
          	// 在插槽表中取出这个group的key与当前composable的key进行对比
            if (writer.groupKey(current) == compositionLocalMapKey &&
                writer.groupObjectKey(current) == compositionLocalMap
            ) {
              	// 返回指定位置的 CompositionLocalMap
                return writer.groupAux(current) as CompositionLocalMap
            }
          	// 死循环，不断的向上找，如果当前组里没有，就继续向上找，直到找到可以与当前匹配的
            current = writer.parent(current)
        }
    }
  	//如果当前composable的slotTable内的数组不为空
    if (slotTable.groupsSize > 0) {
      	//从当前插槽中取里当前离 composable 最近的graoup
        var current = reader.parent
      	// 默认值为-1，如果存在，即意味着存在可组合项
        while (current > 0) {
            if (reader.groupKey(current) == compositionLocalMapKey &&
                reader.groupObjectKey(current) == compositionLocalMap
            ) {
              	//从providerUpdates数组中获取当前CompositionLocalMap，插入见 - startProviders
                return providerUpdates[current]
                    ?: reader.groupAux(current) as CompositionLocalMap
            }
            current = reader.parent(current)
        }
    }
  	//如果没找到，则返回父 composable 的 Provider
    return parentProvider
}
```

用于获取离当前 `composable` 最近的 `CompositionLocalMap` 。

#### resolveCompositionLocal()

```kotlin
private fun <T> resolveCompositionLocal(
    key: CompositionLocal<T>,
    scope: CompositionLocalMap
): T = if (scope.contains(key)) {
  	// 如果当前父CompositionLocalMap里包含了当前local,直接从map中取
    scope.getValueOf(key)
} else {
  	// 否则也就意味着其目前就是最顶层，没有父local，直接使用默认值
    key.defaultValueHolder.value
}
```

使用当前 `CompositionLocal` 当做 **key** ,然后去离当前最近的 `CompositionLocalMap` 里查找对应的 **value** ,如果有直接返回，否则使用 `CompositionLocal` 的默认值。

#### 总结

所以当我们使用 `CompositionLocal.current` 获取数据时，内部其实是通过 `currentCompositionLocalScope()` 获取父`CompositionLocalMap`，注意，这里为什么是map呢？ 因为这里获取的是当前父可组合函数下 **所有** 的 `CompositionLocal`,所以源码里的`consume` 方法参数里需要传递当前 `CompositionLocal` 进去，判断当前我们要取的 `local` 有没有存在在其中，如果有，则直接取，否则使用默认值。

那问题来了，**我们的 CompositionLocal是怎么被可组合树保存的呢？** 带着这个问题，我们继续往下深究。



### CompositionLocalProvider

要想知道 `CompositionLocal` 是如何被可组合树保存，必须从下面开始看起。

```kotlin
fun CompositionLocalProvider(vararg values: ProvidedValue<*>, content: @Composable () -> Unit) {
    currentComposer.startProviders(values)
    content()
    currentComposer.endProviders()
}
```

这里的 `currentComposer` 又是什么？而且为什么先start，后end呢？

我们点进去 `currentComposer` 看看。

```kotlin
/**
 * Composer is the interface that is targeted by the Compose Kotlin compiler plugin and used by
 * code generation helpers. It is highly recommended that direct calls these be avoided as the
 * runtime assumes that the calls are generated by the compiler and contain only a minimum amount
 * of state validation.
 */
interface Composer
     👇
internal class ComposerImpl : Composer
```

可以看到在 `Composer` 的定义里, `Composer` 是为 **Compose kotlin** 编译器插件 提供的,google强烈不建议我们自己手动调用，也就是说，这里的 `start` 和 `end` 其实就是两个标记，编译器会自己调用，或者说就是为了方便编译器。接着我们去看 **startProviders()**

#### startProviders

```kotlin
@InternalComposeApi
override fun startProviders(values: Array<out ProvidedValue<*>>) {
  	//获得当前Composable下的CompositionLocal-Map
    val parentScope = currentCompositionLocalScope()
  	...
    val currentProviders = invokeComposableForResult(this) {
        compositionLocalMapOf(values, parentScope)
    }
    val providers: CompositionLocalMap
    val invalid: Boolean
  	//如果可组合项正在插入树中或者其是第一次被调用，则为true
    if (inserting) {
      	//更新当前的CompositionLocalMap
        providers = updateProviderMapGroup(parentScope, currentProviders)
        invalid = false
        hasProvider = true
    } else {
      	//如果当前可组合项不可以跳过(即发生了改变)或者providers不相同，则更新当前Composable的group
        if (!skipping || oldValues != currentProviders) {
            providers = updateProviderMapGroup(parentScope, currentProviders)
            invalid = providers != oldScope
        } else {
          	//否则跳过当前更新
            skipGroup()
            providers = oldScope
            invalid = false
        }
    }
		//如果本次重组无效且没有正在插入，更新当前group的 CompositionLocalMap
    if (invalid && !inserting) {
        providerUpdates[reader.currentGroup] = providers
    }
  	// 将当前的操作push到栈中,后续还要再弹出
    providersInvalidStack.push(providersInvalid.asInt())
    ...
  	// 将providers数据写入group,最终会写入到SlotTable-slots(插槽的缓冲区)数组中
    start(compositionLocalMapKey, compositionLocalMap, false, providers)
}
```

这个方法用于启动数据提供者,如果看过 `compose` 的设计原理就会知道这里其实就相当于 `group` 的一个开始标记，其内部的内容主要是先获取离当前 `composable` 最近的 `CompositionLocalMap` ,然后使用 **compositionLocalMapOf()** 将当前传递进来的 `value` 更新到对应的 `CompositionLocalMap` 中并返回,然后将这个map再更新到当前group中。

相应的我们说了，这是一个开始标记，自然也存在一个终止标记，即 end,在上述源码中，我们可以知道，其就是 **endProviders()**:

#### endProviders

```kotlin
override fun endProviders() {
    endGroup()
    endGroup()
  	// 将当前的操作出栈
    providersInvalid = providersInvalidStack.pop().asBool()
}
```

其作用就是结束提供者的调用，至于为什么 `end` 两次，应该是防止正在写入导致的不一致问题，如果有大佬存在不同理解，就评论区分享一下吧。

### 总结

当我们使用 `CompositionLocalProvider` 将数据绑定到 `CompositionLocal` 时，其内部会将其保存到距离当前 `composable` 最近的 `CompositionLocalMap` 里面，当我们后续想要使用的时候，使用 `CompositionLocal.current` 读取数据时，其会去查找对应的 `CompositionLocalMap` ，并且以我们的 `CompositionLocal` 为 **key**,如果存在则返回，否则使用默认值返回。



## 碎碎念

本文其实并不是特别难的问题，但却是 `Compose` 在实际中会遇到的一个问题，解决这个问题很简单，但理解背后的设计却是更重要的，希望通过本文，大家可以更好的理解 `CompositionLocal` 在实际中的场景以及设计理念。当然了解具体的源码其实还是需要了解 `Compose` 的基础设计，这点参考文章下方贴出的Android开发者链接即可。后续我将继续深追 `Compose` 在实际应用中需要解决的问题并且分析思路，如果有错误，希望不吝赐教。

如果本文对你有所帮助，欢迎点赞支持一下，大家加油 :)



## 参考链接

[官方文档 - 使用 CompositionLocal 将数据的作用域限定在局部](https://developer.android.com/jetpack/compose/compositionlocal)

[Android开发者 - 深入详解 Jetpack Compose | 实现原理](https://mp.weixin.qq.com/s/CENFvRrZy6Tzhr3-FZIeNg) 

[Android开发者 - 深入详解 Jetpack Compose | 优化 UI 构建](https://mp.weixin.qq.com/s?__biz=MzAwODY4OTk2Mg==&mid=2652067315&idx=1&sn=b003ced0f0c86684c5189d31a6a77f92&chksm=808cf5b6b7fb7ca0fb12604578387cf1cf0c7d7d1847499e50b3700c4039c35adb6e806406b9&scene=21#wechat_redirect)

