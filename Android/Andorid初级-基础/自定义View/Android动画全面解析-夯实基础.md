# Android动画全面解析-夯实基础

Android的动画可以分为两种：传统动画与属性动画，如果严格细分的话，可以分为三种，那就是 View动画(补件动画),帧动画，属性动画。

开始之前，我们先准备一些概念：

![我爱编程，编程使我快乐](../../assets\我爱编程，编程使我快乐.jpg)



**View动画(补件动画)**

View 动画 通过对场景里的对象不断做图像变换(平移-TranslateAnimation，缩放-ScaleAnimation，旋转-RotateAnimation，透明度-AlphaAnimation)从而产生动画效果，它是一种渐进式动画，并且View动画支持自定义。

相关的继承关系：

![1562204086265](../../assets\1562204086265.png)



**帧动画**

帧动画通过顺序播放一系列图像从而产生动画效果，可以简单理解为 图片切换 动画，很显然，如果图片过大就会 导致 OOM。

相关的继承关系：

![1562204024171](E:\Android_NoteBook\Android_NoteBook\assets\1562204024171.png)



**属性动画**

与View动画相比，View动画改变的只是View 显示的位置，而没有改变View 的响应区域，属性动画可以对任何对象做动画，甚至还可以没有对象，相应的动画效果也得到了加强，不再像 View 动画那样只能支持 4种简单的变换，同样属性动画框架提供了动画集合类(AnimatorSet)，通过动画集合类可以将多个属性动画以组合的形式显示出来。

相关继承关系：

![1562203934412](E:\Android_NoteBook\Android_NoteBook\assets\1562203934412.png)







好了，介绍了相应的基础概念

下面我们来详细攻略吧！

![嗨嗨，醒醒，敲代码了](E:\Android_NoteBook\Android_NoteBook\assets\嗨嗨，醒醒，敲代码了.jpg)



## View动画：

又称补件动画，可以分为4种形式(旋转,平移，缩放，透明)，一般采用xml文件的形式，原因是代码会更加容易书写和阅读，同时也更容易复用。

首先我们在res 里建一个 **anim** 文件夹，存放这些动画。

### Alpha

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    fromAlpha -> 设置透明度的初始值，其中0.0是透明，1.0是不透明
    toAlpha -> 设置透明度的结束值值，其中0.0是透明，1.0是不透明
    duration  动画持续时间/毫秒
    interpolator  ->时间插值器，根据时间流逝百分比计算动画进度的百分比
-->
<set>
    <alpha xmlns:android="http://schemas.android.com/apk/res/android"
        android:duration="1000"
        android:fromAlpha="1.0"
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:toAlpha="0.0" />
</set>
```

![alpha](E:\Android_NoteBook\Android_NoteBook\assets\alpha.gif)



### Scale

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <!--
        fromXScale  -> 水平方向缩放起始值
        toXScale  ->水平方向缩放结束值

        fromYScale  -> 竖直方向缩放起始值
        toYScale  ->竖直方向缩放结束值

        pivotX  ->缩放轴点的x坐标，它会影响缩放的效果
        pivotY  ->缩放轴点的Y坐标，它会影响缩放的效果
        注意： 如果将坐标设为view的左边界或者右边界，即(0%)
        那么只会向一个方向缩放，默认是按照中心点同时缩放
    -->
    <scale
        android:duration="2000"
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:interpolator="@android:anim/accelerate_interpolator"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0.0"
        android:toYScale="0.0" />
</set>
```

![scale2](http://ww2.sinaimg.cn/large/006tNc79ly1g4w14k623ig30u01hcqv5.gif)

### translate

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--
    fromXDelta 起始点x值
    fromYDelta 起始点Y 值

    toXDelta  结束点 x 值
    toYDelta  结束点 y 值
    -->
    <translate
        android:duration="2000"
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200"
        android:toYDelta="250" />

</set>
```

![translate](E:\Android_NoteBook\Android_NoteBook\assets\translate.gif)



### rotate

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--
        fromDegrees ->旋转开始角度
        toDegrees  ->旋转结束角度
        pivotX  ->旋转轴点的x坐标
        注意：这里的坐标是相对与自己左边界的距离
        1 相对于自己的左边界的距离，单位像素值。（例如 "5"）
        2 相对于自己的左边界的距离与自身宽度的百分比。（例如 "5%"）
        3 相对于父View的左边界的距离与父View宽度的百分比。（例如 "5%p"）

         pivotY  ->旋转轴点的Y坐标
        注意：这里的坐标是相对与自己上边界的距离
        1 相对于自己的上边界的距离，单位像素值。（例如 "5"）
        2 相对于自己的上边界的距离与自身宽度的百分比。（例如 "5%"）
        3 相对于父View的上边界的距离与父View宽度的百分比。（例如 "5%p"）
    -->
    <rotate
        android:duration="2000"
        android:fromDegrees="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="360" />
</set>
```

![rotate](E:\Android_NoteBook\Android_NoteBook\assets\rotate.gif)



### 混合使用：

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000">
    <translate
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200"
        android:toYDelta="200" />
    <rotate
        android:fromDegrees="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="360" />
    <alpha
        android:fromAlpha="0.5"
        android:toAlpha="1.2" />
    <scale
        android:fromXScale="0.3"
        android:fromYScale="0.3"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1.0"
        android:toYScale="1.0" />

</set>
```

![all](E:\Android_NoteBook\Android_NoteBook\assets\all-1562210368303.gif)

**代码里使用：**

```java
//调用处
startAnimation(R.anim.all_anim);

private void startAnimation(int p) {
    Animation alpha = AnimationUtils.loadAnimation(this, p);
    imgGithub.startAnimation(alpha);
}
```





## 帧动画

帧动画我们平时其实没少见，比如京东的快递小哥奔跑的样式，或者咸鱼的等待样式等等。(ps:图片是从网上偷的)

![img](E:\Android_NoteBook\Android_NoteBook\assets\1115031-b52487cb6b97f911.gif)

**嗯，因为找不到素材，所以就将就一下，我就献出常用的表情包供大家 学(kuai) 习(le):**

```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="true">
    <!--
        注意：避免使用尺寸较大的图片，以免OOM
    -->
    <item
        android:drawable="@drawable/happy1"
        android:duration="200" />
    <item
        android:drawable="@drawable/happy2"
        android:duration="200" />
    ...
    <item
        android:drawable="@drawable/happy10"
        android:duration="200" />
</animation-list>
```

![frame](E:\Android_NoteBook\Android_NoteBook\assets\frame.gif)

代码里使用：

```java
imgGithub.setImageResource(R.drawable.frame);
AnimationDrawable animationDrawable = (AnimationDrawable) imgGithub.getDrawable();
animationDrawable.start();
```





## 属性动画

接下来就是重点了，前面几种可能都是 小菜一碟，那么接下来的就要认真学习啦！

![要开车了哦](E:\Android_NoteBook\Android_NoteBook\assets\要开车了哦.jpg)

抱歉，带错图了(bu ke neng)。不过没关系，先上车再说。



属性动画，顾名思义它是对于对象属性的动画，因此，所有view动画的内容，都可以通过属性动画实现，而且，相比View动画，它的效果会更好更强大。***务必请深刻体会这句话，对任何属性都可以动画，前提是拥有set,get***



### 简单入门

我们先用属性动画实现上面透明效果。

```java
  private void AlphaAnimation() {
 ObjectAnimator animator =ObjectAnimator.ofFloat(imgGithub, "alpha", 1.0f, 1.0f, 0.0f, 0.0f);
        animator.setDuration(2000);
        animator.start();
    }
```

![salpha](http://ww1.sinaimg.cn/large/006tNc79ly1g4w14kxmmlg30u01hc4qp.gif)

不知道你们有没有发现，使用属性动画实现的效果，动画从1f -> 0f，最后图形真的不见了，而使用View动画的，最后还会重新显示出来。没有发现的同学，请翻到上面再看一下，就能发现区别。这也就是为什么属性动画更加强大的原因之一，动画结束后的状态是会保存，而不是画面上的变一下而已。

当然，属性动画也可以组合实现。

```java
    private void AllAnimation() {
        //设置透明度 x轴，y轴透明度 从0f ->1f
        ObjectAnimator alpha = ObjectAnimator.ofFloat(imgGithub, "alpha", 0f, 0f, 1f, 1f);
        //x轴参数 缩放0f -> 1f
        ObjectAnimator scaleX = ObjectAnimator.ofFloat(imgGithub, "scaleX", 0.0f,1.0f);
        //y轴参数 缩放0f -> 1f
        ObjectAnimator scaleY = ObjectAnimator.ofFloat(imgGithub, "scaleY", 0.0f,2.0f);
        //x轴参数 位置，从100 -> 400
        ObjectAnimator transX = ObjectAnimator.ofFloat(imgGithub, "translationX", 300, 400);
        //y轴参数 位置，从100 -> 200
        ObjectAnimator transY = ObjectAnimator.ofFloat(imgGithub, "translationY", 100,200);
        //旋转起始角度与终止角度
        ObjectAnimator rotation = ObjectAnimator.ofFloat(imgGithub, "rotation", 0,360);
        AnimatorSet set=new AnimatorSet();
        //按顺序播放动画
        //set.playSequentially(alpha,transX,transY,rotation);
        //同时启动所有动画
        set.playTogether(alpha,transX,transY,rotation);
        //设置动画时间
        set.setDuration(2000);
        set.start();
    }
```

相应的效果就不放图了。

除了用代码实现以外，还可以通过 xml 来定义。属性动画需要定义在res/animator 目录下，但是我个人非常不建议在xml中使用属性动画，除了麻烦以外，如果我们要移动一个相应的View,因为我们无法知道屏幕的大小，那么xml就很难满足我们的需要。所以，这里就不演示相应的xml了，这方面感兴趣的同学可以自行百度学习。



### 插值器与估值器：

***这个是我们要掌握的重点.***

**TimeInterpolator** 中文翻译为**时间插值器**，它的作用是**根据时间流逝的百分比来计算出当前属性值改变的百分比**，系统预置的有 LinearInterpolator(线性插值器：均速动画)，AccelerateDecelerateInterpolator (加速减速插值器：动画两头慢中间快)和 Decelerate-Interpolator(减速插值器：动画越来越慢)等。***也就是说，它是用来控制动画快慢节奏的。***

TypeEvaluator 为 类型估值算法，也叫**估值器**，它的作用是**根据当前属性改变的百分比来计算改变后的属性值**，系统预置的有 IntEvaluator(针对整型属性)，FloatEvaluator(针对浮点型属性) 和 ArgbEvaluator(针对Color属性)。 ***也就是说，******它决定了动画如何从初始值过渡到结束值。***

理解插值器(Interpolator) 和 估值器(TypeEvaluator) 很重要，它们是实现非匀速动画的重要手段。

上面这样说，肯定有点涩口,所以我们还是用张图来看。

![1562226127196](http://ww4.sinaimg.cn/large/006tNc79ly1g4w14lera1j30gz046wfi.jpg)

如上图(来源于Android官方文档)所示，它表示一个匀速动画，采用 线性插值器 和整型估值算法，在 40ms内，View 的x属性实现 从0到 40的变换。

由于动画的刷新率为 10ms/帧，所以该动画将分为5帧进行，在第三帧的时候(x=20,t=20ms)，也就是当时间t=20ms时，时间的流逝百分比是 0.5,意味着现在时间过了一半，那么X 应该改变多少呢？这个就由插值器和 估值算法来确定。以 线性插值器来看，当时间流逝一半的时候，x的变换 也应该是一半，即x 的改变是 0.5，为什么呢？因为它是匀速动画。

下面查看它的源码：

```java
@HasNativeInterpolator
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {

    public LinearInterpolator() {
    }

    public LinearInterpolator(Context context, AttributeSet attrs) {
    }

    public float getInterpolation(float input) {
        return input;
    }

    /** @hide */
    @Override
    public long createNativeInterpolator() {
        return NativeInterpolatorFactoryHelper.createLinearInterpolator();
    }
}
```

很显然，我们可以发现，线性插值器的输入值与返回值相同，因此插值器返回值是0.5，这意味着 x的改变是0.5，这个时候插值器的工作就完成了。具体 x 变成了什么值，需要估值算法来确定。我们来看一下 整型估值算法的源码：

```java
public class IntEvaluator implements TypeEvaluator<Integer> {
    public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
        int startInt = startValue;
        return (int)(startInt + fraction * (endValue - startInt));
    }
}
```

上述算法中 ，evalute 的三个参数分别表示估值小数，开始值和结束值，对应于我们的例子分别就是 0.5,0,40.根据上述算法，整型估值返回给我们的结果是20，这就是(x=20,t=20ms)的由来。

属性动画要求对象的该属性有 set 和 get 方法(可选)。插值器和估值算法除了系统提供的以外，我们还可以自定义。实现方式也很简单，因为插值器和估值算法都是一个接口，且内部都只有一个方法，我们只需要派生一个类实现接口就可以了。然后就可以做出各种动画效果了。具体来说就是： **自定义插值器需要实现 Interpolator 或者 TimeInterpolator，自定义估值算法需要实现 TypeEvaluator。另外就是如果要对其他类型（非int，float,Color）做动画，那么必须要自定义类型估值算法。**

***本来这里不想写这么多比较生涩的概念，但是有些概念还是要知道的好，这样也还为我们下面的Demo做好充足的预习工作。***



### 属性动画的原理：

我们接下来分析一下属性动画的原理：

属性动画要求动画作用的对象提供该属性的 get 和 set方法，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用 set 方法，每次传递给 set 方法的值都不一样，确切来说是随着时间推移，所传递的值越来越接近最终值。所以，要想让一个动画生效，必须同时满足以下两个条件：

1. 必须提供相应的 set方法，如果动画的时候没有传递初始值，那么还要提供 get 方法，因为系统要去获取 属性的初始值；
2. 相应的 set 方法 对属性所做的改变必须能够通过某种方法反映出来，比如会带来UI 改变之类，否则将无动画效果。



## 属性动画-实践：

根据上面所总结的经验，我们先来做一个简单的Demo:

我们使用属性动画放大一个Button.

```java
ObjectAnimator.ofInt(btnZheng, "width", 100,500).setDuration(1000).start();
```

![btn](http://ww1.sinaimg.cn/large/006tNc79ly1g4w14lydmrg30u01hc7ia.gif)

就这么一句代码，就可以实现Button 从100dp到500dp的变化，是不是很神奇。在以前的Android版本中，如果我们要动态更改Button的宽度等属性，是一件比较麻烦的事，因为我们没有相应的设置方法。而最近几年，google已经提供了相应的设置方法,无疑是给我们减少了很多工作。

我们日常开发中，还有很多场景并没有相应的 set方法，这个时候我们就可以按照下面这三种方法来实现需求。

**有以下几种方法：**

> 1. **给你的对象加上 get 和 set方法，前提是你拥有权限。暂时抛开这个不看**
> 2. **用一个类来包装原始对象，间接为其提供 get 和 set方法。**
> 3. **采用 ValueAnimator ，监听动画过程，自己实现属性的改变。(这个自定义View用的挺多)**

其实，我们所有的自定义View，都不约而同继承于View，那么相应的set属性它已经拥有了，所以我们做的无非就是让属性动画去驱动，我们所需要的就是一个变化的值。



### 方法2 -实例测试：

我们这次来动态的改变ImageView的大小。因为ImageView 本身并没有自带 setWidth()方法，刚好满足我们方法2的要求。

先新建一个包装类：

```java
public class ImageViewapper {
    private ImageView image;
    public ImageViewapper(ImageView image) {
        this.image = image;
    }

    public int getHeight(){
        return image.getLayoutParams().height;
    }

    public void setHeight(int height){
        image.getLayoutParams().height=height;
        image.requestLayout();
    }
}
```

代码里调用

```java
ImageViewapper viewapper=new ImageViewapper(imgGithub);
ObjectAnimator.ofInt(viewapper, "height", 0,500).setDuration(1000).start();
```

看一下效果：

![bao](http://ww3.sinaimg.cn/large/006tNc79ly1g4w14mg114g30u01hcx0d.gif)

**是不是也实现了相应的效果。**





#### 方法3-实践：

方法3我们常用于自定义View，所以我们画一个小圆来测试一下效果，先上代码。

CircleView

```java
public class CircleView extends View { 
    private int width;
    private int height;
    //画笔
    private Paint paint;
    public CircleView(Context context) {
        super(context);
        //初始化画笔
        initPaint();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        width=w/2;
        height=h/2;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(width,height);
        canvas.drawCircle(0,0,200,paint);
    }
    private void initPaint(){
        paint=new Paint();
        paint.setColor(Color.GREEN);
        paint.setStyle(Paint.Style.FILL);
        paint.setStrokeWidth(3f);
    }
}
```

xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:id="@+id/lin_view"
    android:layout_height="match_parent"
    tools:context=".animation.AnimActivity"
    android:orientation="horizontal">
</LinearLayout>
```

Activity

```java
public class AnimActivity extends AppCompatActivity {

    private LinearLayout linView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_anim);
        initView();
        CircleView circleView=new CircleView(this);
        linView.addView(circleView);
    }

    private void initView() {
        linView = (LinearLayout) findViewById(R.id.lin_view);
    }
}
```

我们先用代码将view引入，便于接下来的测试，先看看开始的效果：

![1562233006001](http://ww3.sinaimg.cn/large/006tNc79ly1g4w14mrqetj30b10ix3yi.jpg)

因为View本身就自带了 setAlpha方法，所以我们可以直接调用属性动画的方法，这点和上面的放大Button，差别并不是很大。

```java
ObjectAnimator.ofFloat(circleView,"alpha",0f,1f).setDuration(5000).start();
```

![cricle](http://ww3.sinaimg.cn/large/006tNc79ly1g4w14nu9g3g30u01hck6i.gif)



接下来我们用上面说的方法三实现一下：

修改CircleView代码：

```java
public class CircleView extends View {
    private int width;
    private int height;
    private Paint paint;
    //透明进度
    private float currentValue=0f;

    public CircleView(Context context) {
        super(context);
        initPaint();
        startAnim();
    }


    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        width=w/2;
        height=h/2;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(width,height);
        canvas.drawCircle(0,0,200,paint);
        setAlpha(currentValue);
    }
    private void initPaint(){
        paint=new Paint();
        paint.setColor(Color.GREEN);
        paint.setStyle(Paint.Style.FILL);
        paint.setStrokeWidth(3f);
    }
    private void startAnim(){
        ValueAnimator valueAnimator=ValueAnimator.ofFloat(0f,1f);
        valueAnimator.addUpdateListener(animation -> {
            //获得当前动画进度
            currentValue = (float) animation.getAnimatedValue();
            //刷新布局 -> ui线程使用,会调用onDraw方法重新绘制
            invalidate();
        });
        //设置动画时间并启动
        valueAnimator.setDuration(5000).start();
    }
}
```

**上面代码中我们使用了 ValueAnimator 来监听动画的过程，但是ValueAnimator 本身不作用于任何对象，也就是说直接使用它没有任何动画效果。它可以对一个值做动画，然后我们监听其动画过程，动态的更改我们对象中的属性值，这样也就相当于对我们的view做了动画。**

效果都是一样的，所以我们就没必要带图了。



#### AnimatorSet

```java
  ObjectAnimator animator1 = ObjectAnimator.ofFloat(animationBt,"translationX",300);
  animator1.setDuration(500);
  ObjectAnimator animator2 = ObjectAnimator.ofFloat(animationBt,"scaleX",0.5f);
  animator2.setDuration(500);
  ObjectAnimator animator3 = ObjectAnimator.ofFloat(animationBt,"scaleY",0.5f);
  animator3.setDuration(500);
  
  AnimatorSet set = new AnimatorSet();
 
 
  //1.按先后顺序执行动画
  set.setDuration(1500);
  set.playSequentially(animator1,animator2,animator3);
  set.start();
 
 
  //2.一起执行动画
  set.setDuration(500);
  set.playTogether(animator1,animator2,animator3);
  set.start();
 
 
  //3.先在500ms完成第一个，在一起完成第2个和第3个
  set.setDuration(1000);
  set.play(animator2).with(animator3);
  set.play(animator1).before(animator2);   
  set.start();  
```



## 注意事项：

通过动画可以实现一些非常好看的效果，使用过程中，掌握基本的优化，无论是对我们开发还时程序的健壮性都是有很大的提高的，所以接下来，我们总结以下使用过程中应该注意的事项：

1. OOM问题

   这个问题主要出现于帧动画中，当图片数量过多并且图片较大时就非常容易出现这个问题。所以实际开发中需要尽量避免使用帧动画。

2. 内存泄漏

   在属性动画中有一类无限循环的动画，这类动画需要在Activity 退出时及时停止，否则将导致 Activity将无法释放从而导致内存泄漏，然而View动画不存在此问题。所以好的习惯是使用完的动画应及时释放相关资源。

3. View动画的问题

   View动画是对View的影像做动画，并不是真正改变View的状态，因此有时候回出现动画完成后 View 无法隐藏的问题，即 setVisibility(View.Gone) 失效了，这个时候只要调用 view.clearAnimation() 清除View 动画即可解决问题。

4. 不要使用 px

   在进行动画的过程中，要尽量使用 dp,使用 px 会导致 在不同的设备上有不同的效果。

5. 动画元素的交互

   将 view 移动后，在view 动画中，view的事件响应位置依然在原位置，在属性动画中，位置为动画结束后的位置。

6. 硬件加速

   使用动画的过程中，建议开启硬件加速，这样会提高动画的流畅度。



到了这里，Aandroid的动画基本就讲完了，其中特别关注的就是属性动画了，下一篇我会对属性动画的工作原理从源码层分析一下。本篇如果有帮到你的地方，不胜荣幸。



![教我写代码](http://ww4.sinaimg.cn/large/006tNc79ly1g4w14opht7j305h045mx0.jpg)

**更多Android开发知识请访问—— [Android开发日常笔记](https://github.com/Petterpx/Android_NoteBook)，欢迎Star,你的小小点赞，是对我的莫大鼓励。**

参阅：

- Android开发艺术探索


