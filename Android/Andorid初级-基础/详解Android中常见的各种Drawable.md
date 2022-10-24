# 由浅入深、详解Android中Drawable的工程化实践



## 引言

对于 `Drawable` ，一直没有专门记录，日常开发中，也是属于忘记了再搜一下。主要是使用程度有限(仅仅只是`shape`或者 `layer` 等冰山一角)，另一方面是 `Android` 对其的高度抽象，导致从没去关注过细节，从而对于 `Drawable` 没有真正的理解其设计与存在的意义。

反而是偶尔一次发现其他同学的运用，才明白了自己的狭隘，为此，怀着无比惭愧的心情，写下此篇，与君共勉。

鉴于此，本篇将完整的描述开发中常见各种 `Drawable` ,以及如何更好的运用,本篇难度较低，不涉及源码，适合轻松阅读。

## 来者何人

2022的今天,随便问一个Android开发，`Drawable` 是什么？

> 比如我。他(她)肯定会告诉你(鄙视的眼神)，你si不si傻，`Drawable` 都不知道，`Drawable`,`Drawble`，`Drawable`不就是...😐
>
> 不就是经常用来设置的图像吗？🫣(不确定语气,似乎说的不完整)

上述说的有错吗，也没错。嗯，但总觉得差点什么,过于简单？细心的你肯定会觉得没这么简单。

**那到底什么是Drawable?**

> `Drawable` 表示的是一种可以在Canvas上进行绘制的抽象概念。人话就是 就是指可在屏幕上绘制的图形。

就这？就这？就这？

这说了个啥，水水水，一天就知道水文章？🫵🏼

嗯🧐，在开发角度来看，`Drawable` 是一个抽象的类，用来表示可以绘制在屏幕上绘制的图形。我们常见有很多种 `Drawable` ,比如Bitmapxx,Colorxxx,Shapexxx，**它们一般都用于表示图像，但严格上来说，又不全是图像**。

![表情包：你是不是外面有狗了？](https://tva1.sinaimg.cn/large/e6c9d24ely1h6ca3ij61hj20690683yf.jpg)

**后半句用人话怎么理解呢？**

> 对于普通的图形或图片，我们肯定没法更改，因为其已经固定了(资源文件)。
>
> 但是对于 `Drawable`，虽然某种程度上也是图形(矢量资源)，但其具备处理或绘制具体显示逻辑的方式。也就是说，这是一个支持修改的图形，比如我们可以把一张图塞给了 `BitmapDrawable` ,但依然可以做二次调整，比如拉伸一下，改一下位置，给这张图上再添加点别的什么东西。或者也可以理解为这是一个简化版的View，只不过它更简易，目的纯粹。其无法像 `View` 一样接收事件或者和用户交互，其更像一个绘制板，指哪打哪，仅作为显示使用。
>

当然除了简单的绘图，`Drawable` 还提供了很多通用api,使得我们可以与正在绘制的内容进行交互，从而更加完善。

相应的，`Drawable` 内部其实也有自己的宽高、通过 `intrinsicWidth`、`intrinsicHeight` 即可获取。需要注意的是：

- `Drawable` 的宽高不等于其展示时的大小，我们可以认为 `Drawable` 不存在大小的概念，因为其用于View背景时，其会被拉伸至View的同等大小。
- 也并不是所有的 `Drawable` 都有内部宽高,比如，由一个图片所形成的 `Drawable` ，其相应的宽高也就是图片的宽高,而由颜色所形成的`Drawable` ,相应的内部也不存在宽高。

## Drawable的种类

如下所示，Drawable有如下类型：

![image-20220919224205510](https://tva1.sinaimg.cn/large/e6c9d24ely1h6canm2c6uj21f20lwq82.jpg)

> 好家伙，这也太多了吧，而且后续还会越来越多。

当然这么多，我们一般其实根本不可能全部用上，常见的有如下几种类别：

### 无状态

- **BitmapDrawable**

  `<<bitmap`

  用于将图片转为BitmapDrawable;

- **ShapeDrawable**

  `<<shape`

  通过颜色来构造Drawable;

- **VectorDrawable**

  `<<vector`

  矢量图，Android5.0及以上支持。便于在缩放过程中保证显示质量，以及一个矢量图支持多个屏幕，减少apk大小;

- **TransitionDrawable**

  `<<transition`

  用于实现Drawable间的淡入淡出效果;

- **InsetDrawable**

  `<<inset`

  用于将其他Drawable内嵌到自己当中,并可以在四周留出一定的间距。当一个View希望自己的背景比实际的区域小时，可以采用其来实现。

### 有状态

- **StateListDrawable**

  `<<selector`

  用于有状态交互时的View设置，比如 **按下时** 的背景，**松开时** 的背景，有焦点时的背景灯；

- **LevelListDrawable**

  `<<level-list`

  根据等级(level)来切换不同的 Drawble。在View中可以通过设置 setImageLevel 更改不同的drawable;

- **ScaleDrawable**

  `<<scale`

  根据不同的等级(level)指定 Drawable 缩放到一定比例;

- **ClipDrwable**

  `<<clip`

  根据当前等级(`level`)来裁剪 `Drawable`;

## 常见的Drawable

### BitmapDrawable

**常见使用场景**

用于表示一张图片，用于设置 `bitmap` 在 `BitmapDrawable` 区域内的绘制方式时使用，如水平平铺或者竖直平铺以及扩展铺满。

xml中的标签：**<bitmap>**

**常见的属性有如下：**

- **android:src** 

  资源id

- **android:antialias**

  开启图片抗锯齿，用于让图片变得平滑，同时抗锯齿也会一定程度上降低图片清晰度，不过幅度几乎无法感知；

- **android:dither**

  开启抖动效果，为低像素机型做的自动降级显示，保证显示效果。比如当前图片彩色模式为ARGB8888,而设备屏幕色彩模式为RGB555，此时开启抖动就可以避免图片显示失真；

- **android:filter**

  过滤效果。在图片尺寸被拉伸或者压缩时，有助于保持显示效果；

- **android:gravity**

  当前图片小于容器尺寸时，此选项便于对图片进行定位,当titleMode开启时，此属性将失效；

- **android:mipMap**

  纹理映射开关，主要是为了应对图片大小缩放处理时，Android可以通过纹理映射技术提前按缩小的层级生成图片预存储在内存中，以此来提高速度与图片质量。默认情况下，mipmap文件夹里的默认开启此开关，drawable默认关闭。但需要注意，此开关只能建议系统开启此功能，至于最终是否真正开启，取决于系统。

- **android:tileMode**

  用于设置图片的平铺模式，有以下几个值：[`disabled`、`clamp`、`repeat`、`mirror`]

  - `disabled` (默认值) 关闭平铺模式
  - `clamp`  图片四周的像素会扩展到周围区域
  - `repeat` 水平和竖直方向上的平铺效果
  - `mirror` 在水平和竖直方向的的镜面投影效果

![image-20220922231438984](https://tva1.sinaimg.cn/large/e6c9d24ely1h6fsgggowej211n0i9wiv.jpg)

**示例代码：**

```kotlin
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.ic_doge)
val drawable = BitmapDrawable(bitmap).apply {
    setTileModeXY(Shader.TileMode.CLAMP, Shader.TileMode.CLAMP)
    isFilterBitmap = true
    gravity = Gravity.CENTER
    setMipMap(true)
    setDither(true)
}
ivDrawable.background = drawable
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:dither="true"
    android:filter="true"
    android:gravity="center"
    android:mipMap="true"
    android:src="@drawable/test"
    android:tileMode="repeat" />
```

---

### ShapeDrawable

**常见使用场景**

通过颜色来构造图形，作为背景描边或者背景色渐变时使用，可以说是最常见的一种 `Drawable`。

xml中的标签：**<shape>**

**常见的属性如下：**

- **shape**

  表示图形的形状，如下四个选项：`rectangle`(矩形)、`oval`(椭圆)、`line`(横线)、`ring`(圆环)

- **corners**

  表示shape的四个角的角度，只适用于矩形shape。

  - android:`radius` 为四个角设置相同的角度
  - android:`topLeftRadius` 设置左上角角度
  - android:`bottomLeftRadius` 设置左下角角度
  - android:`bottomRightRadius` 设定右下角的角度

- **gradient**

  用于表示渐变效果，与 <solid> 标签互斥(其表示纯色填充)

  - android:`angle` 渐变的角度，默认为0,其值必须为45的倍数, **0表示从左向右，90表示从下到上**
  - android:`centerX` 渐变中心点的横坐标
  - android:`centerY` 渐变中心点纵坐标
  - android:`startColor` 渐变的起始色
  - android:`centerColor` 渐变的中间点
  - android:`endColor` 渐变的结束色
  - android:`gradientRadius` 渐变半径,仅当android:type=“radial”时有效
  - android:`useLevel` 是否使用等级区分，在 `StateListDrawable` 时有效，默认 **false**
  - android:`type` 渐变类型，`linear`(线性渐变)、`radial`(径向渐变)、`sweep`
  
- **solid** 表示纯色填充

- **stroke** 用于设置描边

  - android:`width` 描边宽度
  - android:`color`  描边颜色
  - android:`dashWidth` 描边虚线时的宽度
  - android:`dashGap` 描边虚线时的间隔

  ![image-20220925193303529](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j2wthcm0j21nc0l6ju1.jpg)

- **padding**

  用于表示空白，其代表了在View中使用时的空白。但其在shape中并没有什么作用，可以在 `layer-list` 中进行使用。

- **size**

  用于表示 `shape` 的 **固有大小** ，但其不代表shape最终显示的大小。因为对于shape来说，其没有宽/高的概念，因为其最终被设置到View上作为背景时,其会被自动拉伸或缩放。但作为drawable,它拥有着固有宽/高,即通过 `getIntrinsicWidth`，`getIntrinsicHeight` 获取。对于某些Drawable而言，比如BitMapDrawable时，其宽高就是图片大小；而对于shape时，其就不存在大小，默认为-1。当然你也可以通过 size 设置大小，但其最终代表的是shape的固有宽高，作为背景时其并不是最终显示时的宽高。

**示例如下：**

![image-20220925201141661](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j40zgfzkj21b20mlgp4.jpg)

---

### LayerDrawable

表示一种层次化的集合 `drawable` ,一般常见于需要多个 `Drawable` **叠加** 摆放效果时使用。

一个 `layer-list` 中可以包含多个 item ,每个item表示一个 `Drawable` ,其常用的属性 android:`top`,`left`,`right`,`bottom` 。相当于相对View的 **上下左右** 偏移量，单位为像素。此外也可以通过 `Drawable` 引用一个已有的 `Drwable` 资源。

xml中的标签：**<layer-list>**

示例如下:

![image-20220925201808649](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j47p27cdj21op0u0n0v.jpg)

---

### StateListDrawable

用于为不同的 **View状态** 引用不同的 `Drawable` ,比如在View **按下** 时，View **禁用** 时等。

xml中的标签：**<selector>**

**常用的属性如下：**

- **constantSize**

  表示其固有大小是否随着状态而改变。

  因为每次切换状态时，都会伴随着 `Drawable` 的改变，如果此时不是用于背景，则如果 `Drawable` 的固有大小不一致，则会导致`StateListDrawable` 的大小发生变化。如果此值为 **true** ,则表示当前 `StateDrawable` 的固有大小是当前其内部所有 `Drawable` 中最大值。反之，则根据状态决定;

- **android:dither**

  是否开启抖动效果，用于在低质量屏幕上获得较好的显示效果;

- **variablePadding**

  表示padding是否随着状态而改变，true表示跟随状态而决定，取决于当前显示的drawable,false则选取drawable集合中padding最大值。

示例如下：
| <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6j5fqhrvzj216n0u0djf.jpg" alt="image-20220925210028927" style="zoom:50%;" /> | <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6j59574ddg20u01pob29.gif" alt="statelaist" style="zoom:33%;" /> |
| :----------------------------------------------------------: | ------------------------------------------------------------ |

------

### LevelListDrawable

用于根据不同的等级表示一个 `Drawable` 集合。

默认等级范围为0,最小为0,最大为10000，可以在View中使用 `Drawable` 从而设置相应的 **level** 切换不同的 `Drawable`。如果这个drawable被用于ImageView 的 **前景**Drawable,还可以通过 ImageView.setImageViewLevel 来切换。

xml中的标签：**<level-list>**

示例代码如下：

![image-20220925210620317](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j5lu2mxfj21500i6gnx.jpg)

在代码中即可通过 setLevel切换。

```kotlin
 view.background.level = 10
 view.background.level = 200
```

---

### TransitaionDrawable

用于实现两个 `Drawable` 之间的淡入淡出效果。

xml中的标签：**<transition>**

示例如下：

| ![image-20220925213450558](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j6fhivooj20ql0dwdh8.jpg) | <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6j6b2h8cyg20u01pob29.gif" alt="translition" style="zoom:25%;" /> |
| :----------------------------------------------------------: | ------------------------------------------------------------ |

---

### InsetDrawable

用于将其他 `Drawable` 内嵌到自己当中，并可以在四周留出一定的间距。比如当某个 `View` 希望自己的背景比自己的实际区域小时，可以采用这个 `Drawable` ,当然采用 `LayerDrawable` 也可以实现。

xml中的标签：**<inset>**

其属性分别如下：

- **android:inset** 表示四边内凹大小
- **android:insetTop** 表示顶部内凹大小
- **android:insetLeft** 表示左边内凹大小
- **android:insetBottom** 表示底部内凹大小
- **android:insetRight** 表示右边内凹大小

![image-20220925215737375](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j736xwoaj20vi0b7750.jpg)

---

### ScaleDrawable

用于根据等级(`level`)将指定的 `Drawable` 缩放到一定比例。

xml中的标签：**<scale>**

**相应的属性如下所示：**

- **android:scaleGravity** 

  类似于与android:gravity

- **android:scaleWidth**

  指定宽度的缩放比例(相对于原drawable缩放了多少)

- **android:scaleHeight**

  指定高度的缩放比例(相对于原drawable缩放了多少)

- **android:level**(minSdk>=`24`)

  指定缩放等级,默认为0,即最小,最高10000,此值越大,最终显示的drawable越大

> 需要注意的是，当level为0时，其不会显示，所以我们使用ScaleDrawable时，需要在代码中，将 drawable.level 调为1。

示例如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/level2_drawable"
    android:level="1"
    android:scaleWidth="70%"
    android:scaleHeight="70%"
    android:scaleGravity="center" />
```

---

### ClipDrawable

用于根据当前等级(level)来裁剪另一个 `Drawable`。

xml中的标签：**<clip>**

具体属性如下：

- **android:drawable**

  需要裁剪的drawable

- **android:clipOrientation**

  裁剪方向,有水平(horizontal)、竖直(vertical) 两种

- **android:level**(minSdk>=`24`)

  设置等级，为0时表示完全裁剪(即隐藏)，值越大，裁剪的越小。最大值10000(即不裁剪,原图)。

- **android:gravity**

  | 参数              | 含义                                                         |
  | ----------------- | ------------------------------------------------------------ |
  | top               | 内部drawable位于容器顶部，不改变大小。ps: 竖直裁剪时，则从底部开始裁剪。 |
  | bottom            | 内部drawable位于容器底部，不改变大小。ps: 竖直裁剪时，则从顶部开始裁剪。 |
  | left(默认值)      | 内部drawable位于容器底部，不改变大小。ps: 水平裁剪时，则从顶部开始裁剪。 |
  | right             | 内部drawable位于容器右边，不改变大小。ps: 水平裁剪时，从右边开始裁剪。 |
  | start             | 同left                                                       |
  | end               | 同right                                                      |
  | center            | 使内部drawable在容器中居中，不改变大小。 ps:竖直裁剪时，从上下同时开始裁剪；水平裁剪时，从左右同时开始。 |
  | center_horizontal | 内部的drawable在容器中水平居中，不改变大小。 ps:水平裁剪时，从左右两边同时开始裁剪。 |
  | center_vertical   | 内部的drawable在容器中垂直居中，不改变大小。 ps:竖直裁剪时，从上下两边同时开始裁剪。 |
  | fill              | 使内部drawable填充满容器。 ps:仅当level为0(0表示ClipDrawable被完全裁剪，即不可见)时,才具有裁剪行为。 |
  | fill_horizontal   | 使内部drawable在水平方向填充容器。 ps:如果水平裁剪，仅当level为0时，才具有裁剪行为。 |
  | fill_vertical     | 使内部drawable在竖直方向填充容器。 ps:如果垂直裁剪，仅当level为0时，才具有裁剪行为。 |
  | clip_horizontal   | 竖直方向裁剪。                                               |
  | clip_vertical     | 竖直方向裁剪。                                               |

**示例如下：**

![image-20220926214129677](https://tva1.sinaimg.cn/large/e6c9d24ely1h6kc8pq1zsj210l0bjt9w.jpg)

![image-20220926213734053](https://tva1.sinaimg.cn/large/e6c9d24ely1h6kc4o4a44j211q0nwgpb.jpg)

---

## 自定义Drawable

通常情况下，我们往往用不到自定义 `Drawable` ,主要源于Android已经提供了很多通常会用到的功能，不过了解自定义 `Drawable` 在某些场景下可以非常便于我们开发体验。

自定义 `Drawable` 也很简单，我们只需要继承 `Drawable` 即可，从而实现：

- **draw()**

  实现自定义的绘制。

  如果要获取当前 `Drawable` 绘制的边界大小，可以通过 **getBounds()** 获取；

  如果需要获取当前 `Drawable` 的中心点，也可以通过 **getBounds().exactCenterX()** ,或者 **getBounds().centerX()** ,区别在于前者用于获取精确位置；

- **setAlpha()**

  设置透明度；

- **setColorFilter()**

  设置滤镜效果；

- **getOpacity()**

  返回当前 `Drawable` 的透明度；

比如画一个类似的 `ProgressBar` ,因为其是一个 `Drawable` ,所以可以用作任意的 `View`。

```kotlin
class CustomCircleProgressDrawable : Drawable(), Animatable {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val rectF = RectF()
    private var progress = 0F
    private val valueAnimator by lazy(LazyThreadSafetyMode.NONE) {
        ValueAnimator.ofFloat(0f, 1f).apply {
            duration = 2000
            repeatCount = Animation.INFINITE
            interpolator = LinearInterpolator()
            addUpdateListener {
                progress = it.animatedValue as Float
                invalidateSelf()
            }
        }
    }

    init {
        paint.style = Paint.Style.STROKE
        paint.strokeWidth = 10f
        paint.strokeCap = Paint.Cap.ROUND
        paint.color = Color.GRAY
        start()
    }

    override fun draw(canvas: Canvas) {
        var min = (min(bounds.bottom, bounds.right) / 2).toFloat()
        paint.strokeWidth = min / 10
        min -= paint.strokeWidth * 3
        val centerX = bounds.exactCenterX()
        val centerY = bounds.exactCenterY()
        rectF.left = centerX - min
        rectF.right = centerX + min
        rectF.top = centerY - min
        rectF.bottom = centerY + min
        paint.color = Color.GRAY
        canvas.drawArc(rectF, -90f, 360f, false, paint)
        paint.color = Color.GREEN
        canvas.rotate(360F * progress, centerX, centerY)
        canvas.drawArc(rectF, -90F, 30F + 330F * progress, false, paint)
    }

    override fun setAlpha(alpha: Int) {
        paint.alpha = alpha
        invalidateSelf()
    }

    override fun setColorFilter(colorFilter: ColorFilter?) {
        paint.colorFilter = colorFilter
        invalidateSelf()
    }

    override fun getOpacity(): Int {
        return PixelFormat.TRANSLUCENT
    }

    override fun start() {
        if (valueAnimator.isRunning) return
        valueAnimator.start()
    }

    override fun stop() {
        if (valueAnimator.isRunning) valueAnimator.cancel()
    }

    override fun isRunning(): Boolean {
        return valueAnimator.isRunning
    }
}
```

原理也很简单，我们实现了 `onDraw` 方法，在其中利用 `canvas` 绘制了两个圆环,其中前者是作为背景，后者不断地利用属性动画进行变化，并且不断旋转 `canvas` ，从而实现类似进度条的效果。

效果如下：

![Kapture 2022-09-28 at 23.07.28](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/Kapture%202022-09-28%20at%2023.07.28.gif)

## 实践推荐

比如我们现在有这样一个 `View` ,需要在左上角展示一个文字,背景是张图片，另外还有一个从顶部到下的半透明渐变阴影。

如下图所示：

![image-20220927235824306](https://tva1.sinaimg.cn/large/e6c9d24ely1h6lltiimn2j20ci0bo0t5.jpg)

一般情况下，我们肯定会不假思索的写出如下代码。

![image-20220928234411154](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/image-20220928234411154.png)

上述写法没有问题，但其并不是所有场景的最推荐方式。比如这种样式此时需要在 `RecyclerView` 中展示呢？

所以，此时就可以利用 `Drawable` 简化我们的 `View` 层级，改造思路如下：

![image-20220929101205068](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291012134.png)

如上所示，相对于最开始，我们将布局层级由 3 层降低为了 1 层,对于性能的提升也将指数级上升。

现在有同学可能要问了，你的这个 `View` 很简单，自定义一个 `Drawable` 还好说，那 `View` 很复杂呢？难不成我真要纯纯自定义吗？

要回答这个问题，其实很简单，我们要明确 `Drawable` 的意义，其只是一个**可绘制的图像** 。过于复杂的View,我们可以将其拆分为多个层级，然后对于其中纯展示的View,使用 `Drawable` 降低其复杂度。

**从某个角度来说，Drawable也可以是我们自定义View的好帮手。**

## 

## 总结

合理利用 `Drawable` 会很大程度提高我们的使用体验。相应的，对于布局优化，`Drawable` 也是一大利器。问题回到文章最开始，如果现在再问你。`Drawable` 到底是什么? 如何自定义一个 `Drawable` ? 如何利用其做一些骚操作？我想，这都不是问题。

## 参考

- Android艺术探索 - Android中的Drawable
- [Android开发者 - 可绘制资源](https://developer.android.com/guide/topics/resources/drawable-resource)

## 关于我

> 我是 Petterp ,一个Android工程师，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！













# 由浅入深、详解Android中Drawable的工程化实践



## 引言

对于 `Drawable` ，一直没有专门记录，日常开发中，也是属于忘记了再搜一下。主要是使用程度有限(仅仅只是`shape`或者 `layer` 等冰山一角)，另一方面是 `Android` 对其的高度抽象，导致从没去关注过细节，从而对于 `Drawable` 没有真正的理解其设计与存在的意义。

反而是偶尔一次发现其他同学的运用，才明白了自己的狭隘，为此，怀着无比惭愧的心情，写下此篇，与君共勉。

鉴于此，本篇将完整的描述开发中常见各种 `Drawable` ,以及如何更好的运用,本篇难度较低，不涉及源码，适合轻松阅读。

## 来者何人

2022的今天,随便问一个Android开发，`Drawable` 是什么？

> 比如我。他(她)肯定会告诉你(鄙视的眼神)，你si不si傻，`Drawable` 都不知道，`Drawable`,`Drawble`，`Drawable`不就是...😐
>
> 不就是经常用来设置的图像吗？🫣(不确定语气,似乎说的不完整)

上述说的有错吗，也没错。嗯，但总觉得差点什么,过于简单？细心的你肯定会觉得没这么简单。

**那到底什么是Drawable?**

> `Drawable` 表示的是一种可以在Canvas上进行绘制的抽象概念。人话就是 就是指可在屏幕上绘制的图形。

就这？就这？就这？

这说了个啥，水水水，一天就知道水文章？🫵🏼

嗯🧐，在开发角度来看，`Drawable` 是一个抽象的类，用来表示可以绘制在屏幕上绘制的图形。我们常见有很多种 `Drawable` ,比如Bitmapxx,Colorxxx,Shapexxx，**它们一般都用于表示图像，但严格上来说，又不全是图像**。

![表情包：你是不是外面有狗了？](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041566.jpg)

**后半句用人话怎么理解呢？**

> 对于普通的图形或图片，我们肯定没法更改，因为其已经固定了(资源文件)。
>
> 但是对于 `Drawable`，虽然某种程度上也是图形(矢量资源)，但其具备处理或绘制具体显示逻辑的方式。也就是说，这是一个支持修改的图形，比如我们可以把一张图塞给了 `BitmapDrawable` ,但依然可以做二次调整，比如拉伸一下，改一下位置，给这张图上再添加点别的什么东西。或者也可以理解为这是一个简化版的View，只不过它更简易，目的纯粹。其无法像 `View` 一样接收事件或者和用户交互，其更像一个绘制板，指哪打哪，仅作为显示使用。
>

当然除了简单的绘图，`Drawable` 还提供了很多通用api,使得我们可以与正在绘制的内容进行交互，从而更加完善。

相应的，`Drawable` 内部其实也有自己的宽高、通过 `intrinsicWidth`、`intrinsicHeight` 即可获取。需要注意的是：

- `Drawable` 的宽高不等于其展示时的大小，我们可以认为 `Drawable` 不存在大小的概念，因为其用于View背景时，其会被拉伸至View的同等大小。
- 也并不是所有的 `Drawable` 都有内部宽高,比如，由一个图片所形成的 `Drawable` ，其相应的宽高也就是图片的宽高,而由颜色所形成的`Drawable` ,相应的内部也不存在宽高。

## Drawable的种类

如下所示，Drawable有如下类型：

![image-20220919224205510](https://tva1.sinaimg.cn/large/e6c9d24ely1h6canm2c6uj21f20lwq82.jpg)

> 好家伙，这也太多了吧，而且后续还会越来越多。

当然这么多，我们一般其实根本不可能全部用上，常见的有如下几种类别：

### 无状态

- BitmapDrawable

  `<<bitmap`

  用于将图片转为BitmapDrawable;

- ShapeDrawable

  `<<shape`

  通过颜色来构造Drawable;

- VectorDrawable

  `<<vector`

  矢量图，Android5.0及以上支持。便于在缩放过程中保证显示质量，以及一个矢量图支持多个屏幕，减少apk大小;

- TransitionDrawable

  `<<transition`

  用于实现Drawable间的淡入淡出效果;

- InsetDrawable

  `<<inset`

  用于将其他Drawable内嵌到自己当中,并可以在四周留出一定的间距。当一个View希望自己的背景比实际的区域小时，可以采用其来实现。

### 有状态

- StateListDrawable

  `<<selector`

  用于有状态交互时的View设置，比如 **按下时** 的背景，**松开时** 的背景，有焦点时的背景灯；

- LevelListDrawable

  `<<level-list`

  根据等级(level)来切换不同的 Drawble。在View中可以通过设置 setImageLevel 更改不同的drawable;

- ScaleDrawable

  `<<scale`

  根据不同的等级(level)指定 Drawable 缩放到一定比例;

- ClipDrwable

  `<<clip`

  根据当前等级(level)来裁剪Drawable;

## 常见的Drawable

### BitmapDrawable

**常见使用场景**

用于表示一张图片，用于设置 `bitmap` 在 `BitmapDrawable` 区域内的绘制方式时使用，如水平平铺或者竖直平铺以及扩展铺满。

xml中的标签：**<bitmap>**

**常见的属性有如下：**

- **android:src** 

  资源id

- **android:antialias**

  开启图片抗锯齿，用于让图片变得平滑，同时抗锯齿也会一定程度上降低图片清晰度，不过幅度几乎无法感知；

- **android:dither**

  开启抖动效果，为低像素机型做的自动降级显示，保证显示效果。比如当前图片彩色模式为ARGB8888,而设备屏幕色彩模式为RGB555，此时开启抖动就可以避免图片显示失真；

- **android:filter**

  过滤效果。在图片尺寸被拉伸或者压缩时，有助于保持显示效果；

- **android:gravity**

  当前图片小于容器尺寸时，此选项便于对图片进行定位,当titleMode开启时，此属性将失效；

- **android:mipMap**

  纹理映射开关，主要是为了应对图片大小缩放处理时，Android可以通过纹理映射技术提前按缩小的层级生成图片预存储在内存中，以此来提高速度与图片质量。默认情况下，mipmap文件夹里的默认开启此开关，drawable默认关闭。但需要注意，此开关只能建议系统开启此功能，至于最终是否真正开启，取决于系统。

- **android:tileMode**

  用于设置图片的平铺模式，有以下几个值：[`disabled`、`clamp`、`repeat`、`mirror`]

  - `disabled` (默认值) 关闭平铺模式
  - `clamp`  图片四周的像素会扩展到周围区域
  - `repeat` 水平和竖直方向上的平铺效果
  - `mirror` 在水平和竖直方向的的镜面投影效果

![image-20220922231438984](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041703.jpg)

**示例代码：**

```kotlin
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.ic_doge)
val drawable = BitmapDrawable(bitmap).apply {
    setTileModeXY(Shader.TileMode.CLAMP, Shader.TileMode.CLAMP)
    isFilterBitmap = true
    gravity = Gravity.CENTER
    setMipMap(true)
    setDither(true)
}
ivDrawable.background = drawable
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:dither="true"
    android:filter="true"
    android:gravity="center"
    android:mipMap="true"
    android:src="@drawable/test"
    android:tileMode="repeat" />
```

---

### ShapeDrawable

**常见使用场景**

通过颜色来构造图形，作为背景描边或者背景色渐变时使用，可以说是最常见的一种 `Drawable`。

xml中的标签：**<shape>**

**常见的属性如下：**

- **shape**

  表示图形的形状，如下四个选项：`rectangle`(矩形)、`oval`(椭圆)、`line`(横线)、`ring`(圆环)

- **corners**

  表示shape的四个角的角度，只适用于矩形shape。

  - android:`radius` 为四个角设置相同的角度
  - android:`topLeftRadius` 设置左上角角度
  - android:`bottomLeftRadius` 设置左下角角度
  - android:`bottomRightRadius` 设定右下角的角度

- **gradient**

  用于表示渐变效果，与 <solid> 标签互斥(其表示纯色填充)

  - android:`angle` 渐变的角度，默认为0,其值必须为45的倍数, **0表示从左向右，90表示从下到上**
  - android:`centerX` 渐变中心点的横坐标
  - android:`centerY` 渐变中心点纵坐标
  - android:`startColor` 渐变的起始色
  - android:`centerColor` 渐变的中间点
  - android:`endColor` 渐变的结束色
  - android:`gradientRadius` 渐变半径,仅当android:type=“radial”时有效
  - android:`useLevel` 是否使用等级区分，在 `StateListDrawable` 时有效，默认 **false**
  - android:`type` 渐变类型，`linear`(线性渐变)、`radial`(径向渐变)、`sweep`
  
- **solid** 表示纯色填充

- **stroke** 用于设置描边

  - android:`width` 描边宽度
  - android:`color`  描边颜色
  - android:`dashWidth` 描边虚线时的宽度
  - android:`dashGap` 描边虚线时的间隔

  ![image-20220925193303529](https://tva1.sinaimg.cn/large/e6c9d24ely1h6j2wthcm0j21nc0l6ju1.jpg)

- **padding**

  用于表示空白，其代表了在View中使用时的空白。但其在shape中并没有什么作用，可以在 `layer-list` 中进行使用。

- **size**

  用于表示 `shape` 的 **固有大小** ，但其不代表shape最终显示的大小。因为对于shape来说，其没有宽/高的概念，因为其最终被设置到View上作为背景时,其会被自动拉伸或缩放。但作为drawable,它拥有着固有宽/高,即通过 `getIntrinsicWidth`，`getIntrinsicHeight` 获取。对于某些Drawable而言，比如BitMapDrawable时，其宽高就是图片大小；而对于shape时，其就不存在大小，默认为-1。当然你也可以通过 size 设置大小，但其最终代表的是shape的固有宽高，作为背景时其并不是最终显示时的宽高。

**示例如下：**

![image-20220925201141661](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041120.jpg)

---

### LayerDrawable

表示一种层次化的集合 `drawable` ,一般常见于需要多个 `Drawable` **叠加** 摆放效果时使用。

一个 `layer-list` 中可以包含多个 item ,每个item表示一个 `Drawable` ,其常用的属性 android:`top`,`left`,`right`,`bottom` 。相当于相对View的 **上下左右** 偏移量，单位为像素。此外也可以通过 `Drawable` 引用一个已有的 `Drwable` 资源。

xml中的标签：**<layer-list>**

示例如下:

![image-20220925201808649](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041188.jpg)

---

### StateListDrawable

用于为不同的 **View状态** 引用不同的 `Drawable` ,比如在View **按下** 时，View **禁用** 时等。

xml中的标签：**<selector>**

**常用的属性如下：**

- **constantSize**

  表示其固有大小是否随着状态而改变。

  因为每次切换状态时，都会伴随着 `Drawable` 的改变，如果此时不是用于背景，则如果 `Drawable` 的固有大小不一致，则会导致`StateListDrawable` 的大小发生变化。如果此值为 **true** ,则表示当前 `StateDrawable` 的固有大小是当前其内部所有 `Drawable` 中最大值。反之，则根据状态决定;

- **android:dither**

  是否开启抖动效果，用于在低质量屏幕上获得较好的显示效果;

- **variablePadding**

  表示padding是否随着状态而改变，true表示跟随状态而决定，取决于当前显示的drawable,false则选取drawable集合中padding最大值。

示例如下：
| <img src="https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041756.jpg" alt="image-20220925210028927" style="zoom:50%;" /> | <img src="https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041570.gif" alt="statelaist" style="zoom:33%;" /> |
| :----------------------------------------------------------: | ------------------------------------------------------------ |

------

### LevelListDrawable

用于根据不同的等级表示一个 `Drawable` 集合。

默认等级范围为0,最小为0,最大为10000，可以在View中使用 `Drawable` 从而设置相应的 **level** 切换不同的 `Drawable`。如果这个drawable被用于ImageView 的 **前景**Drawable,还可以通过 ImageView.setImageViewLevel 来切换。

xml中的标签：**<level-list>**

示例代码如下：

![image-20220925210620317](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041626.jpg)

在代码中即可通过 setLevel切换。

```kotlin
 view.background.level = 10
 view.background.level = 200
```

---

### TransitaionDrawable

用于实现两个 `Drawable` 之间的淡入淡出效果。

xml中的标签：**<transition>**

示例如下：

| ![image-20220925213450558](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041902.jpg) | <img src="https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041636.gif" alt="translition" style="zoom:25%;" /> |
| :----------------------------------------------------------: | ------------------------------------------------------------ |

---

### InsetDrawable

用于将其他 `Drawable` 内嵌到自己当中，并可以在四周留出一定的间距。比如当某个 `View` 希望自己的背景比自己的实际区域小时，可以采用这个 `Drawable` ,当然采用 `LayerDrawable` 也可以实现。

xml中的标签：**<inset>**

其属性分别如下：

- **android:inset** 表示四边内凹大小
- **android:insetTop** 表示顶部内凹大小
- **android:insetLeft** 表示左边内凹大小
- **android:insetBottom** 表示底部内凹大小
- **android:insetRight** 表示右边内凹大小

![image-20220925215737375](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041100.jpg)

---

### ScaleDrawable

用于根据等级(`level`)将指定的 `Drawable` 缩放到一定比例。

xml中的标签：**<scale>**

**相应的属性如下所示：**

- **android:scaleGravity** 

  类似于与android:gravity

- **android:scaleWidth**

  指定宽度的缩放比例(相对于原drawable缩放了多少)

- **android:scaleHeight**

  指定高度的缩放比例(相对于原drawable缩放了多少)

- **android:level**(minSdk>=`24`)

  指定缩放等级,默认为0,即最小,最高10000,此值越大,最终显示的drawable越大

> 需要注意的是，当level为0时，其不会显示，所以我们使用ScaleDrawable时，需要在代码中，将 drawable.level 调为1。

示例如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/level2_drawable"
    android:level="1"
    android:scaleWidth="70%"
    android:scaleHeight="70%"
    android:scaleGravity="center" />
```

---

### ClipDrawable

用于根据当前等级(level)来裁剪另一个 `Drawable`。

xml中的标签：**<clip>**

具体属性如下：

- **android:drawable**

  需要裁剪的drawable

- **android:clipOrientation**

  裁剪方向,有水平(horizontal)、竖直(vertical) 两种

- **android:level**(minSdk>=`24`)

  设置等级，为0时表示完全裁剪(即隐藏)，值越大，裁剪的越小。最大值10000(即不裁剪,原图)。

- **android:gravity**

  | 参数              | 含义                                                         |
  | ----------------- | ------------------------------------------------------------ |
  | top               | 内部drawable位于容器顶部，不改变大小。ps: 竖直裁剪时，则从底部开始裁剪。 |
  | bottom            | 内部drawable位于容器底部，不改变大小。ps: 竖直裁剪时，则从顶部开始裁剪。 |
  | left(默认值)      | 内部drawable位于容器底部，不改变大小。ps: 水平裁剪时，则从顶部开始裁剪。 |
  | right             | 内部drawable位于容器右边，不改变大小。ps: 水平裁剪时，从右边开始裁剪。 |
  | start             | 同left                                                       |
  | end               | 同right                                                      |
  | center            | 使内部drawable在容器中居中，不改变大小。 ps:竖直裁剪时，从上下同时开始裁剪；水平裁剪时，从左右同时开始。 |
  | center_horizontal | 内部的drawable在容器中水平居中，不改变大小。 ps:水平裁剪时，从左右两边同时开始裁剪。 |
  | center_vertical   | 内部的drawable在容器中垂直居中，不改变大小。 ps:竖直裁剪时，从上下两边同时开始裁剪。 |
  | fill              | 使内部drawable填充满容器。 ps:仅当level为0(0表示ClipDrawable被完全裁剪，即不可见)时,才具有裁剪行为。 |
  | fill_horizontal   | 使内部drawable在水平方向填充容器。 ps:如果水平裁剪，仅当level为0时，才具有裁剪行为。 |
  | fill_vertical     | 使内部drawable在竖直方向填充容器。 ps:如果垂直裁剪，仅当level为0时，才具有裁剪行为。 |
  | clip_horizontal   | 竖直方向裁剪。                                               |
  | clip_vertical     | 竖直方向裁剪。                                               |

**示例如下：**

![image-20220926214129677](https://tva1.sinaimg.cn/large/e6c9d24ely1h6kc8pq1zsj210l0bjt9w.jpg)

![image-20220926213734053](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041208.jpg)

---

## 自定义Drawable

通常情况下，我们往往用不到自定义 `Drawable` ,主要源于Android已经提供了很多通常会用到的功能，不过了解自定义 `Drawable` 在某些场景下可以非常便于我们开发体验。

自定义 `Drawable` 也很简单，我们只需要继承 `Drawable` 即可，从而实现：

- **draw()**

  实现自定义的绘制。

  如果要获取当前 `Drawable` 绘制的边界大小，可以通过 **getBounds()** 获取；

  如果需要获取当前 `Drawable` 的中心点，也可以通过 **getBounds().exactCenterX()** ,或者 **getBounds().centerX()** ,区别在于前者用于获取精确位置；

- **setAlpha()**

  设置透明度；

- **setColorFilter()**

  设置滤镜效果；

- **getOpacity()**

  返回当前 `Drawable` 的透明度；

比如画一个类似的 `ProgressBar` ,因为其是一个 `Drawable` ,所以可以用作任意的 `View`。

```kotlin
class CustomCircleProgressDrawable : Drawable(), Animatable {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val rectF = RectF()
    private var progress = 0F
    private val valueAnimator by lazy(LazyThreadSafetyMode.NONE) {
        ValueAnimator.ofFloat(0f, 1f).apply {
            duration = 2000
            repeatCount = Animation.INFINITE
            interpolator = LinearInterpolator()
            addUpdateListener {
                progress = it.animatedValue as Float
                invalidateSelf()
            }
        }
    }

    init {
        paint.style = Paint.Style.STROKE
        paint.strokeWidth = 10f
        paint.strokeCap = Paint.Cap.ROUND
        paint.color = Color.GRAY
        start()
    }

    override fun draw(canvas: Canvas) {
        var min = (min(bounds.bottom, bounds.right) / 2).toFloat()
        paint.strokeWidth = min / 10
        min -= paint.strokeWidth * 3
        val centerX = bounds.exactCenterX()
        val centerY = bounds.exactCenterY()
        rectF.left = centerX - min
        rectF.right = centerX + min
        rectF.top = centerY - min
        rectF.bottom = centerY + min
        paint.color = Color.GRAY
        canvas.drawArc(rectF, -90f, 360f, false, paint)
        paint.color = Color.GREEN
        canvas.rotate(360F * progress, centerX, centerY)
        canvas.drawArc(rectF, -90F, 30F + 330F * progress, false, paint)
    }

    override fun setAlpha(alpha: Int) {
        paint.alpha = alpha
        invalidateSelf()
    }

    override fun setColorFilter(colorFilter: ColorFilter?) {
        paint.colorFilter = colorFilter
        invalidateSelf()
    }

    override fun getOpacity(): Int {
        return PixelFormat.TRANSLUCENT
    }

    override fun start() {
        if (valueAnimator.isRunning) return
        valueAnimator.start()
    }

    override fun stop() {
        if (valueAnimator.isRunning) valueAnimator.cancel()
    }

    override fun isRunning(): Boolean {
        return valueAnimator.isRunning
    }
}
```

原理也很简单，我们实现了 `onDraw` 方法，在其中利用 `canvas` 绘制了两个圆环,其中前者是作为背景，后者不断地利用属性动画进行变化，并且不断旋转 `canvas` ，从而实现类似进度条的效果。

效果如下：

![Kapture 2022-09-28 at 23.07.28](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041654.gif)

## 实践推荐

比如我们现在有这样一个 `View` ,需要在左上角展示一个文字,背景是张图片，另外还有一个从顶部到下的半透明渐变阴影。

如下图所示：

![image-20220927235824306](https://tva1.sinaimg.cn/large/e6c9d24ely1h6lltiimn2j20ci0bo0t5.jpg)

一般情况下，我们肯定会不假思索的写出如下代码。

![image-20220928234411154](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291041480.png)

上述写法没有问题，但其并不是所有场景的最推荐方式。比如这种样式此时需要在 `RecyclerView` 中展示呢？

所以，此时就可以利用 `Drawable` 简化我们的 `View` 层级，改造思路如下：

![image-20220929101205068](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202209291012134.png)

如上所示，相对于最开始，我们将布局层级由 3 层降低为了 1 层,对于性能的提升也将指数级上升。

现在有同学可能要问了，你的这个 `View` 很简单，自定义一个 `Drawable` 还好说，那 `View` 很复杂呢？难不成我真要纯纯自定义吗？

要回答这个问题，其实很简单，我们要明确 `Drawable` 的意义，其只是一个**可绘制的图像** 。过于复杂的View,我们可以将其拆分为多个层级，然后对于其中纯展示的View,使用 `Drawable` 降低其复杂度。

**从某个角度来说，Drawable也可以是我们自定义View的好帮手。**

## 总结

合理利用 `Drawable` 会很大程度提高我们的使用体验。相应的，对于布局优化，`Drawable` 也是一大利器。问题回到文章最开始，如果现在再问你。`Drawable` 到底是什么? 如何自定义一个 `Drawable` ? 如何利用其做一些骚操作？我想，这都不是问题。

## 参考

- Android艺术探索 - Android中的Drawable
- [Android开发者 - 可绘制资源](https://developer.android.com/guide/topics/resources/drawable-resource)

## 关于我

> 我是 Petterp ,一个Android工程师，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！

 
