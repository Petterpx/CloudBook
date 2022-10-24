# Compose实践



## Compose中的常用组件

### Column

垂直列表,类似于Android中的LinearLayout.具体示例如下：

属性如下：

```kotlin
Column(
  	//修饰符
    modifier: Modifier = Modifier,
  	//垂直排列方向
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
  	//水平排列方向
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
  	//具体的内容
    content: @Composable ColumnScope.() -> Unit
)
```

![image-20210311214807976](https://tva1.sinaimg.cn/large/008eGmZEgy1gogaxtiii3j322c0u012z.jpg)



对于 `verticalArrangement` 与  `horizontalAlignment` ，除了提供的通用的 **Start,End,Top** 等等 ，你还可以通过如下方式自定义具体的内部view摆放位置。

#### Arrangement

> **用于指定布局子项在主轴方向的排列。**

Arrangement.aligned(BiasAlignment.Vertical(xx))

其中 [0] 表示居中，[1] 表示在底部 ,[-1] 表示在顶部 ,[-1,1]范围之内则表示自定义的区间

```
//垂直方向靠左显示
Arrangement.aligned(BiasAlignment.Vertical(-1f))
//垂直方向居中显示
Arrangement.aligned(BiasAlignment.Vertical(0F))
//垂直方向靠右显示
Arrangement.aligned(BiasAlignment.Vertical(1f))
```





#### BiasAlignment

> 由偏移量指定的[Alignment]：例如，偏移量-1表示与起始位置的对齐，偏移量0表示居中，偏移量1表示末尾。可以指定任何值以获得对齐方式。在[-1，1]范围内，获得的对齐方式会将对齐的尺寸完全定位在可用空间内，而在范围之外，它将对齐的尺寸部分或完全定位在外部。



#### Divider

分割线





#### rember

用于缓存数据，避免数据被重复初始化

```kotlin
setContent {
    var name by remember {
        mutableStateOf("1231")
    }
    Text(text = name)
    lifecycleScope.launch {
        delay(1000)
      	//重新赋值时，上面的mutableStateOf会使用先前初始化的对象
        name = "1212331pp"
    }
}
```

**remember(value)**

带key的缓存,只有value改变后，底下的Text()才会重新执行

```kotlin
@Composable
fun showCharCount(value: String) {
    val length = remember(value) {
        value.length
    }
    Text(text = length.toString())
}
```





#### Compose的状态

非直接无(内部)状态，可以有(内部)状态。

State Hoisting 



#### mutableStateOf()

> 便于数据更改后可以通知使用的之处

#### mutableStateListOf

> 便于数据更改后可以通知使用的之处

#### mutabStateMapOf

> 维护一个map，便于数据更改后通知使用之处



#### 优化被动view Compose



Compose如何知道数据改变？

> Structral Equality: = =  || java equals
>
> Referential Equality: = = =



当对一个对象做改变时：

```kotlin
data class User(var name: String = "petterp")
```

> 当使用var时，compose 将跳过内部name的判断，只有重新替换了新的user的对象，才会引发重组。

> 当使用 val 时，compose 每一次都会判断内部的值是否改变,哪怕更改的对象是同一个对象，但只要内部的数据改变，都会引发重组。

**示例**

```kotlin
data class User(var name: String = "petterp")

@Composable
fun TestTv(d: User) {
      Log.e("petterp", "测试执行--${d.name}")
}

compose
val user = User("petterp")
val user2 = User("petterp12")
var tempUser = mutableStateOf(user)
setContent {
    Column {
        TestTv(tempUser.value)
        Button(
            onClick = {
                tempUser.value = user2
            }
        ) {
            Text(text = "btn1")
        }
        Button(
            onClick = {
                user2.name = "已更改"
            }
        ) {
            Text(text = "btn2")
        }
    }
}
```

先点击btn1,再点击btn2，再点击btn1

点击后结果分别如下:

**var:**

> 测试执行--petterp12





#### @Stable

用于保证某个类是稳定的,便于compose不直接跳过，如果是var就不跳过。



## Modifier

modifier是可以嵌套的。

如下代码所示:

```kotlin
Modifier.background(Color.Blue)
```

**background**

```kotlin
fun Modifier.background(
    color: Color,
    shape: Shape = RectangleShape
) = this.then(
    Background(
        color = color,
        shape = shape,
        inspectorInfo = debugInspectorInfo {
            name = "background"
            value = color
            properties["color"] = color
            properties["shape"] = shape
        }
    )
)
```

**this.then**

```kotlin
infix fun then(other: Modifier): Modifier =
    if (other === Modifier) this else CombinedModifier(this, other)
```

**CombinedModifier**

```kotlin
class CombinedModifier(
    private val outer: Modifier,
    private val inner: Modifier
)
```

#### 业务开发中仿照Modifier的实现可以这样写

```kotlin
interface IListener {

    infix fun then(obj: () -> Unit) {
        if (this !== IListener) obj.invoke()
    }

    companion object : IListener
}

fun test(iListener: IListener = IListener) {
    iListener.then {
    }
}
```

如果某个类经常被用作默认参数，那么可以使用一个静态的实现，以避免多次初始化。



#### foldIn

先加入的先应用

#### foldOn

后面加入的先应用

  

- Offset()

- Offset(offset:Density().)

- fillMaxSize()  填满父控件，=math_parent

- fillMaxHeight()

- wrapContentSize() 忽略父控件对子控件大小限制。比如父modifier给了80，加了这个，子就能超过80

- defaultMinSize() 一旦使用了defaultMinSize，则使用右侧或者子控件的最小值限制

- scroll() 让组件具有滑动功能

  - horizontalScroll()
  - verticalScroll()
  - textFieldScroll

- zIndex(zIndex:Float) 改变绘制顺序，默认从左往右

- graphicsLayer(block:GraphicsKayerScope.()->Unit)

  **GraphicsKayerScope**

  - alpha() 透明度
  - clip() 切割
  - clipToBounds()  
  - rotate(degress:Float) 旋转
  - scale(scaleX:Float,scaleY:Float) 放缩
  - shadow() 阴影
  - toolingGraphinsLayer()  测试使用

  **graphicsLayer**

  > 上述的方法合集。
  >
  > Modifer.graphicsLayer(
  >
  > ​	scaleX:Float=1f,
  >
  > ​	scaleY:Float=1f,
  >
  > ​	alpha:Float=1f
  >
  > ​	...
  >
  > )

- layout() 定义测量与布局，简称自定义View

- paint()  用来画图，一般不使用，使用其下面具体实现 

  - Image()
  - Icon()

- backGround() 背景颜色或者简单的背景规则

- indication() 给当前显示内容增加标记

  - @Compose Button()其实现了，比如波纹效果

- drawBehind()  自定义背景

- drawWithCache() 给控件背景上面画东西

  - border()  边缘线

- onSizeChanged() 重新测量时回调

- onGloballyPositioned()  测量回调

#### ParentDataModifier

- RowScope.Modifier.weight()/ColumnScope.Modifier.weight()
- layoutId() 内部测量时，



onFoucusEvent() 焦点改变时调用







## 动画

> - 声明式是直接操作，而不会去先拿到对象
> - 我们对于动画过程的设置也是隐式的

#### animateDpAsState()

设置属性的初始值，让属性以动画的形式去变化到目标值，适合默认初始化的，而非经常变动的

```kotlin
val size by animateDpAsState(80.dp)  //State
```

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsij04zspwj31560la0xp.jpg" alt="image-20210716091935645" style="zoom:50%;" />

#### snapTo()

直接设置到相应的值，而不经过动画时间

#### animateTo()

缓慢的过渡，而非直接设置。

```kotlin
val anim = remember {
     Animatable(48.dp, Dp.VectorConverter)
 }
 LaunchedEffect(big) {
     anim.snapTo(if (big) 144.dp else 0.dp)
   	 //tween()设置时长
     anim.animateTo(if (big) 96.dp else 48.dp, tween(2000))
 }
```



#### LaunchedEffect

适用于Compose的协程

```kotlin
val anim = Animatable(48.dp, Dp.VectorConverter)
LaunchedEffect(true) {
    anim.animateTo(96.dp)
}
```

##### 注意点

当LaunchedEffect内部的key为基本类型时，将不会再次被改变，即永远只会被调用一次。

##### 实现原理

```kotlin
fun LaunchedEffect(
    key1: Any?,
    block: suspend CoroutineScope.() -> Unit
) {
    val applyContext = currentComposer.applyCoroutineContext
    remember(key1) { LaunchedEffectImpl(applyContext, block) }
}
```

```kotlin
internal class LaunchedEffectImpl(
    parentCoroutineContext: CoroutineContext,
    private val task: suspend CoroutineScope.() -> Unit
) : RememberObserver {
    private val scope = CoroutineScope(parentCoroutineContext)
    private var job: Job? = null

    override fun onRemembered() {
        job?.cancel("Old job was still running!")
        job = scope.launch(block = task)
    }

    override fun onForgotten() {
        job?.cancel()
        job = null
    }

    override fun onAbandoned() {
        job?.cancel()
        job = null
    }
}
```

 

#### updateTransition()

便于同时处理多个动画，并且不会再重组中被多次初始化

```kotlin
var big by remember {
    mutableStateOf(false)
}
val bigTransition = updateTransition(big, label = "更新动画")
val size by bigTransition.animateDp(label = "") { if (it) 100.dp else 50.dp }
val cornerSize by bigTransition.animateDp { if (it) 16.dp else 0.dp }
```



#### Modifier.animateContentSize()



#### Crossfade()



#### AnimatedEddect





## 自定义View

#### Modifier.drawWithContent

按顺序绘制自定义的布局

```kotlin
Text(
    text = "haha",
    Modifier.drawWithContent {
      	//先绘制一个矩形颜色
        drawRect(Color.Green)
        //执行text的content
        drawContent()
    }
)
```

#### Modifier.drawBehind

专门用于绘制背景,compose会将内部的代码全部放到 drawContent 之前。



#### Modifier.drawWithCache

对绘制过程中的对象进行缓存。

```KOTLIN
Modifier.drawWithCache {
    val path = Path()
    // 准备工作
    onDrawWithContent {
        // 绘制背景
        drawContent()
        // 绘制前景
        drawPath(path, Color.Green)
    }

    onDrawBehind {
        drawPath(path, Color.Blue)
    }
}
```

#### Canvas

便于在Compose中直接自定义布局。

```kotlin
Canvas(modifier = Modifier) {
    drawPath()
    drawRect()
}

Image(
    painter = painterResource(id = R.drawable.abc_vector_test),
    modifier = Modifier.drawWithContent {
                drawContent()
                //绘制自定义代码
    },contentDescription = ""
)
```

  



#### Modifier.layout

用于layout的摆放以及控制内部组件的偏移

> constraints 内部组件的限制
>
> measurable 用于修改内部组件的内容与偏移

```kotlin
@Composable
fun TestPadding() =
    Modifier.background(Color.Green).layout { measurable, constraints ->
        val padding = 8.dp.toPx().roundToInt()
        val paddedConstraints = constraints.copy().apply {
            constrainWidth(maxWidth - padding * 2)
            constrainHeight(maxHeight - padding * 2)
        }
        // 测量完后，可以摆放
        val placeable = measurable.measure(paddedConstraints)
        layout(placeable.width + padding * 2, placeable.height + padding * 2) {
            placeable.placeRelative(padding, padding)
        }
    }
```



#### MeasurePolicy

提供测量与布局算法的类

##### 自定义简易column

```kotlin
@Composable
fun CustomColumn(modifier: Modifier = Modifier, content: @Composable () -> Unit) {
    Layout(content, modifier) { measurable, constraints ->
        val parcelables = mutableListOf<Placeable>()
        var width = 0
        var height = 0
        measurable.forEach {
            val placeable = it.measure(constraints)
            height += placeable.height
            width = max(width, placeable.measuredWidth)
            parcelables.add(placeable)
        }
        layout(width, height) {
            var currentHeight = 0
            parcelables.forEach {
                it.placeRelative(0, currentHeight)
                currentHeight += it.height
            }
        }
    }
}
```



#### Instrinsic

固有特性测量,先自己测量一次大概的高度，最终再进行测量。

 -> 如果我不给你限制，你会多大

 -> 如果我限制你的宽为xxx,那么你会是多高 / 如果我限制你的高为xxx,那么你会是多宽

 -> 如果我限制你的宽为 xxx,那么你会最多 / 最少会是多高 ； 如果我限制你的高为xxxx,那么你最多/最少会是多宽

##### 示例

```kotlin
// 内部的divider就不会占满屏幕高度，将与Text对齐
Row(Modifier.height(IntrinsicSize.Max)) {
    Text(text = "1231")
    Divider(Modifier.width(1.dp).fillMaxHeight())
    Text(text = "1231")
}
```



##### 自定义流式布局

```kotlin
@Composable
fun CustomFlowLayout(modifier: Modifier = Modifier, content: @Composable () -> Unit) {
    Layout(content, modifier) { measurtable, constraints ->
        val parcelables = mutableListOf<Placeable>()
        val widths = mutableListOf<Pair<Int, Int>>()
        var maxTempHeight = 0
        var width = 0
        var height = 0
        val maxWidth = constraints.maxWidth
        measurtable.forEach {
            val placeable = it.measure(constraints)
            if (width + placeable.width <= maxWidth) {
                widths.add(width to height)
                maxTempHeight = max(maxTempHeight, placeable.height)
                width += placeable.width
            } else {
                height += maxTempHeight
                widths.add(0 to height)
                width = placeable.width
                maxTempHeight = placeable.height
            }
            parcelables.add(placeable)
        }
        height += maxTempHeight
        layout(maxWidth, height) {
            parcelables.forEachIndexed { index, placeable ->
                placeable.placeRelative(widths[index].first, widths[index].second)
            }
        }
    }
}
```

#### 自定义触摸反馈





## 与原生View的交互

Compose中写view

```
setContent {
    Column {
        Text(text = "1231")
        Text(text = "1231")
        AndroidView({
            //这里只会更新一次，适用于初始化一次
            View(this@MainActivity)
        }, Modifier.size(100.dp)) {
        		//每次 Recompose都会执行
            it.setBackgroundColor(android.graphics.Color.BLACK)
        }
    }
}
```
