# Android-View相关重构

> 关于View的这些你都知道吗，这是View相关部分的一份重构回顾。

### 什么是View?

View是Android中所有控件的基类，无论是Button和TextView,还是LinearLayout或者ConstraintLayout,它们的共同基类都是View.

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggalltijnsj311i0q0abw.jpg" alt="image-20200630204106235" style="zoom:50%;" />

它们之间的关系如上图所示，我们很简单的可以理解为你中有我，我中有你，类似递归一样。





### View的位置参数

View的位置坐标是由它的四个顶点来决定，分别对应于View的四个属性：**top**,**left**,**right**,**bottom**，它们的关系可以用下图来表示。

其中 **top** 是左上角，**left** 是左上角横坐标，**right**是右下角横坐标，**bottom** 是右下角纵坐标。需要注意的是，这些坐标都是相对于View 的父容器来说的。因此它是一种相对坐标，View的坐标和父容器如图所示：

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggannj5btbj315g0q8gou.jpg" alt="image-20200630215158710" style="zoom:50%;" />

上图中我们发现有 **translationX,translationY**两个参数，这两个参数又是什么呢？

这两个参数其实就是view相对于之前的位置偏移量，当我们的View移动(属性动画)后，相应的translation就会改变，left.top这些属性不会改变。

如下例所示：

#### 方法1

```java
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) button.getLayoutParams();
params.leftMargin +=300;
button.requestLayout();
```

这种方法并不会影响 **translationX，但是left发生了改变

#### 方法2

```kotlin
ObjectAnimator.ofFloat(btnLeft, "translationX", 0f, 100f).setDuration(2000).start();
            thread {
                Thread.sleep(3000)
                Log.e("petterp", "xxx" + viewLeft.translationX)
                Log.e("petterp", "yyy" + viewLeft.translationY)
            }
```

我们用属性动画，发现translationX的值改变了。

是不是感觉很奇怪？为什么我直接改变位置，translationx没有变化呢？反而属性动画却影响了这个值？

**总结：**

回顾Android的发展，translationX是Android3.0加入的，随着一起加入的还有属性动画，我们不难猜测，translationX 就是用于保存view移动后的临时坐标。因为我们都知道，属性动画不像view动画这种，在结束后，view的点击事件仍在原处，而是会跟随移动到动画结束位置。然而我们在使用属性动画的时候，view的源坐标是不改变的，改变的只是translationX和x,所以为了便于开发者的更好使用，如果断然直接改变原坐标，那么带来的影响就是开发者如果需要移动view回去，就必须手动记录相应的坐标信息。而且，既然是动画，如果动画直接影响到原坐标信息，这样的设计可能并不是最佳。



## MotoionEvent 和 TouchSlop

### MotionEvent

在手指接触屏幕后所产生的一系列事件中，典型的事件类型有如下几种：

- ACTION_DOWN---------手指刚接触屏幕
- ACTION_MOVE --------手指在屏幕上移动
- ACTION_UP----------手指从屏幕上松开的一瞬间

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，考虑如下几种情况：

- 点击屏幕后离开松手，时间序列为 DOWN-> UP
- 点击屏幕滑动一会再松开，时间序列为 DOWN -> MOVE -> ... MOVE->UP/

一般情况下，通过 MotionEvent 对象我们可以得到点击事件发生的 x和 y坐标。为此，系统提供了两组方法： getx/getY 及 getRawX/getRawY 。它们的区别在于， getX/getY返回的是相对于当前View 左上角的x和坐标，而getRawX/getRawY 返回的是相对于手机屏幕左上角的 x和y坐标。



### TouchSlop

TouchSlop 是系统所能识别出的被认为滑动最小距离，也就是说是手指在屏幕滑动时，如果两次滑动之间的距离小于这个常亮，那么系统就不认为你是在进行滑动操作。原因也很简单：滑动的距离太短，这是一个常量，具体的值和设备有关。我们通常通过 **ViewConfiguration.get(this).scaledTouchSlop** 获取，获取到之后，便于我们再处理滑动时，可以利用这个常亮做一些过滤，因此就可以认为用户是否真的滑动。



## VelocityTracker,GestureDetector Scroller

### VelocityTracker

速度追踪，用于追踪手指在滑动过程中的速度，包括水平和竖直方向的速度。它的使用过程如下：

**在view 的onTouchEvent 方法中追踪当前单击事件的速度**

```kotlin
var velocityTracker: VelocityTracker? = null
        view.setOnTouchListener { v, event ->
            when (event.action) {
                MotionEvent.ACTION_DOWN -> {
                  	//初始化
                    velocityTracker = VelocityTracker.obtain()
                }
                MotionEvent.ACTION_MOVE
                -> {
                  	//监听
                    velocityTracker?.apply { 
                        addMovement(event)
                        computeCurrentVelocity(1000)
                        Log.e("petterp", "x---$xVelocity,y--------$yVelocity")
                    }   
                }
                MotionEvent.ACTION_UP
                -> {
                  	//回收资源
                    velocityTracker?.apply { 
                        clear()
                        recycle()
                    }
                }
            }
            true
        }
```

### 

### GestureDetector

手势监测，用于辅助监测用户的单击，滑动，长按，双击等行为，使用方式如下：

```kotlin
 val tem = GestureDetector(this, TapListenerImpl())
        tem.setIsLongpressEnabled(false)
        btn.setOnTouchListener { v, event ->
            tem.onTouchEvent(event)
        }
```

```kotlin
class TapListenerImpl : GestureDetector.OnGestureListener 
```

| 方法名               | 描述                                                         | 所属接口            |
| -------------------- | ------------------------------------------------------------ | ------------------- |
| onDown               | 手指按下时，由一个 ACTION_DOWN触发                           | OnGestureListener   |
| onShowPress          | 手指轻触屏幕，尚未松开或拖动，由一个 ACTION_DOWN触发。**强调的是没有松开或者拖动的状态** | OnGestureListener   |
| onSingleTapUp        | 手指轻触屏幕后松开，伴随着1个 ACTION_UP  而触发，这是单击行为 | OnGestureListener   |
| onScroll             | 手指按下屏幕并拖动，由一个 ACTION_DOWN,多个ACTION_MOVE触发，即为 **拖动行为** | OnGestureListener   |
| onLongPress          | 用户长久按着屏幕不放，即长按时                               | OnGestureListener   |
| onFling              | 用户按下触摸屏，快速滑动后松开，由一个 ACTION_DOWN,多个 ACTION_MOVE和一个 ACTION_UP触发，这是快速滑动行为。 | OnGestureListener   |
| onDoubleTap          | 双击，由两次连续的单击组成，它不可能和 onSingleTapConfirmed共存 | OnDoubleTapListener |
| onSingleTapConfirmed | 严格的单击行为，即如果触发，则不可能再触发别的单击行为，即这只能是单击，而不能是双击中的一次单击。 | OnDoubleTapListener |
| onDoubleTapEvent     | 表示发生了双击行为，在双击的期间，ACTION_DOWN,ACTION_MOVE和 ACTION_UP都会彼此触发此回调 | OnDoubleTapListener |



### Scroller 

弹性滑动对象，用于实现View 的弹性滑动。**其实Scroller 是用于让View内部的子view滑动，可以理解为让View范围内滑动。**

但需要注意的是，Scroller本身并无法让View弹性滑动，它需要和 View的 computScroll 方法配合使用才能共同完成这个功能。

> **一般情况下，Scroller用于ViewGroup**

**示例代码**

```kotlin
class CustomScrollView : LinearLayout {
    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    )

    val scroller = Scroller(context)

    fun smppthScrollTo(x: Int, y: Int, duration: Int) {
      	//执行滑动，其中duration 为滑动时间
        scroller.startScroll(0,0,x,y,duration)
        invalidate()
    }

    override fun computeScroll() {
        //为true代表还在滚动
        if (scroller.computeScrollOffset()) {
          	//改变View位置
            scrollTo(scroller.currX, scroller.currY)
            postInvalidate()
        }
        super.computeScroll()
    }

}

Activity
class MainActivity : AppCompatActivity() {
    @SuppressLint("ClickableViewAccessibility")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        viewLeft.setOnClickListener {
            viewLeft.smppthScrollTo(100, -500,1000)
        }
    }
}
```

**示例gif**

![Jietu20200719-172219](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwenbfrv6g305j0bnqv6.gif)



## View的滑动

### 使用ScrollTo/scrollBy

这两个方法一般用于View的滑动,但需要注意的是，使用scrollTo和 scrollBy来实现View滑动，只能将View的内容进行移动，并不能将View本身进行移动。

**它们之间关系的变换图：**

![image-20200717002822831](https://tva1.sinaimg.cn/large/007S8ZIlly1ggta38gyl7j31d80u07fm.jpg)



### 通过动画滑动

动画本身就是一种渐进的过程，因此如果通过动画来实现滑动默认就具有动画的效果。

**示例代码**

非常简单，平移ImageView到指定位置

```kotlin
 ObjectAnimator.ofFloat(ivImg, "translationX", 0f, 300f).setDuration(1000).start()
```

**示例效果：**

![Jietu20200719-173517](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwf19gprag305j0bn4qq.gif)

**注意**：

这里我们使用的是属性动画，如果使用View动画，那view的位置其实是没有改变，因为View动画是对View的影像做操作，并不能真正改变View的位置参数，包括宽/高，如果希望动画后的状态保留就需要将 fillAfter 属性设置为true,否则动画完成后其结果会消失。





### 改变布局参数进行滑动

通过改变LayoutParams实现效果，这种方式也相对比较常用，但是我们一般不会使用其进行滑动，一般都是用与改变某个view大小。

但其实也非常好理解，如果我们想移动指定的View，只需要更改其的 marginxx,比如想向下移动100px，就增加marginTop-100px即可。

**示例代码**

```kotlin
class MainActivity : AppCompatActivity() {
    @SuppressLint("ClickableViewAccessibility")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        viewLeft.setOnClickListener {
            startMobile(300)
        }
    }
		//为了便于观察，加入属性动画
    private fun startMobile(leftMargin: Int) {
        val valueAnimator = ValueAnimator.ofInt(0, leftMargin)
        valueAnimator.addUpdateListener {
            val layoutParams = ivImg.layoutParams as ViewGroup.MarginLayoutParams
            layoutParams.leftMargin = dp2px(it.animatedValue as Int)
            ivImg.layoutParams = layoutParams
        }
        valueAnimator.duration = 1000
        valueAnimator.start()
    }
}
```

**示例gif**

![Jietu20200719-173517](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwg0qwg2ug305j0bn4qq.gif)



### 三种滑动方式对比

- scrollTo/scrollBy 是View提供的原生方法，其作用是专用于View的滑动，它可以比较方便的实现滑动效果，并且不影响内部元素的单击事件，但是它只能用于滑动View的内容，并不能滑动View本身；
- 通过动画的方式，是我们比较常见的使用手段，而且一般复杂的ue效果往往都是通过动画实现；
- 最后一种直接改变布局的方式，相对来说使用麻烦点，它主要应用于一些具有交互性的View,因为这些View需要和用户交互，直接通过动画实现会存在问题(即使用view动画时),所以这个时候我们可以通过直接改变布局参数的方式实现。



## View的事件分发机制

其实View的分发机制并没有想象中那么复杂，我们可以简单理解为任务的传递与处理上报，老板派了一个任务给总监，总监又将这个任务派给了你，你完成后又将进度传递给了你上司，上司又将结果汇报给老板，这就是一个递归的过程。

而事件分发从技术角度来说，其实就是对MotionEvent的事件分发过程，即当一个 **MotionEvent** 产生了以后，系统需要把这个事件传递给一个具体的View,而这个传递的过程就是分发过程。点击事件的分发过程由以下几个方法共同完成：**dispatchTouchEvent**,**onInterceptTouchEvent** 和 **onTouchEvent**.

 **dispatchTouchEvent(MotionEvent ev)**

> 用来进行事件分发。如果事件能够传递给当前View,那么此方法一定会被调用，返回结果受当前View的 **onTouchEvent** 和 下级View的 **dispatchTouchEvent** 方法的影响，**表示是否消耗当前事件。**

**onInterceptTouchEvent(MotionEvent event)**

> 在上述方法的内部调用，用于判断是否拦截某个事件。如果当前View 拦截了某个事件，那么在同一个事件序列当中，此方法不会被再次调用，**返回结果表示是否拦截当前事件。**

**onTouchEvent(MotionEvent event)**

> 在 **dispathTouchEvent** 方法中调用，用来处理点击事件，返回结果表示**是否消耗当前事件**，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件。



**关于View的事件分发，之前已经做过了相关记录与源码分析，详细的情况请查看其它文章。Android事件分发基础篇-源码进阶篇**



