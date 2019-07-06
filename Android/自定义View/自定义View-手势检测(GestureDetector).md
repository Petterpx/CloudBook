# 自定义View-手势检测(GestureDetector)

***注明:  非常感谢 [gcssloop](www.gcssloop.com) 的博客，以下为我学习时的笔记记录。***



> **注意：学习前本文需要了解一定的事件分发基础，请参阅我的另一篇博文 [Android事件分发(基础篇)全面解析](https://blog.csdn.net/petterp/article/details/94463191)。**

在开发Android手机应用过程中，可能需要对一些手势做出响应，如：单击，双击，长按，滑动，缩放等。这些都是很常见的手势。就拿最简单的来说吧，假如我们需要判断一个控件是否被双击(即在较短的时间内快速的点击两次)，似乎看起来很简单，但仔细考虑起来，要处理的细节问题却不少。例如：

1. **记录点击次数**：为了判断是否被点击超过一次。所以必须记录点击次数。
2. **记录点击事件**：由于双击事件是较快速的点击两次，想点击一次后，过来几分钟再点击一次肯定不能算是双击事件，所以在记录点击次数的同事也要记录上一次的点击时间，我们可以设置本次点击距离上一次时间超过一定时间(例如：超过100ms)就不识别为双击事件。
3. **点击状态重置**：在响应双击事件，或者判断不是双击事件的时候要重置计数器和上一次点击事件。重置即可以点击的时候判断并进行重新设置，也可以使用定时器等超过一定事件后重置状态。

这样分析起来，判断一个双击事件就有很多麻烦的事情了，别说其他的手势了。所以，系统早就封装了一些方法供我们使用。接下来我们就一起来看看它们是如何使用的吧。

## GestureDetector

> GestureDetector 可以使用MotionEvents 监测各种手势和事件。
>
> GestureDetector.OnGestureListener 是一个回调方法，在发生特定的事件时会调用 Listener 中对应的 方法回调。这个类只能用于检测触摸事件的 MotionEvent。
>
> 使用方法：
>
> - 创建一个 GestureDetector 实例
> - 在 onTouchEvent (MotionEvent) 方法中，确保调用 GestureDetector 实例的onTouchEvent(MotionEvent)。回调中定义的方法将在事件发生时执行。
> - 如果侦听 onContextClick（MotionEvent），则必须在 View 的 onGenericMotionEvent（MotionEvent）中调用 GestureDetector OnGenericMotionEvent（MotionEvent）。

GestureDeterctor 本身的方法比较少，使用起来也非常简单，使用分为3步骤：

```java
//创建监听回调
GestureDetector.SimpleOnGestureListener listener=new GestureDetector.SimpleOnGestureListener(){
    @Override
    public boolean onDoubleTap(MotionEvent e) {
        return super.onDoubleTap(e);
    }
};

//创建检测器
final  GestureDetector detector=new GestureDetector(this,listener);

//给监听器设置数据源
btn_t1.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        return detector.onTouchEvent(event);
    }
});
```

接下来我们了解一下 GetureDetector 里面的内容。



### 构造函数

GettureDetector 一共有5中构造函数，但其中有两种已经被废弃了，1种是重复的，所以我们只需要关注其中两种构造函数即可。

| 构造函数                                                     |
| ------------------------------------------------------------ |
| GestureDetector(Context context, GestureDetector.OnGestureListener listener) |
| GestureDetector(Context context, GestureDetector.OnGestureListener listener, Handler handler) |

第一种构造函数里面需要传递两个参数，上下文（Context）和手势监听器(OnGestureListener).

第二种构造函数则需要多传递一个  Handler 作为参数，这个作用主要是为了给 GestureDetector 提供一个 Looper.

在通常情况下是不需要这个 Handler 的，因为它会在内部自动创建一个 Handler 用于处理数据，如果你再主线程中创建GestureDetector,那么它内部创建的Handler 会自动获得主线程的 Looper,然而如果你在 一个 没有创建 Looper 的子线程中创建 GetureDetector 则需要传递一个 带有 Looper 的Handler 给它，否则就会因为无法获取到Looper导致创建失败。

第二种 构造函数使用方式如下(下面是两种在子线程中创建 GestureDetector 的方法)：

```java
// 方式一、在主线程创建 Handler
final Handler handler = new Handler();
new Thread(new Runnable() {
    @Override public void run() {
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();

// 方式二、在子线程创建 Handler，并且指定 Looper
new Thread(new Runnable() {
    @Override public void run() {
        //这里的  Looper也是主线程的Looper，Activity在初始化的时候会创建Handler对象，并初始化Looper
        final Handler handler = new Handler(Looper.getMainLooper());
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();
```

当然了，使用其他创建 Handler 的方式也是可以的，重点传递的 Handler 一定要有 Looper.假如子线程准备了 Looper 那么可以直接使用第一种构造函数进行创建，如下：

```java
new Thread(new Runnable() {
    @Override public void run() {
        Looper.prepare(); // <- 重点在这里
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener());
        // ... 省略其它代码 ...
    }
}).start();
```

### 

### 手势监听器

既然是手势监测，自然需要在对应的手势出现的时候通知调用者，最合适的自然就是事件监听模式，目前 GestureDetecotr 有四种监听器。

| 监听器                                                       | 简介                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [OnContextClickListener](https://developer.android.com/reference/android/view/GestureDetector.OnContextClickListener.html) | 这个很容易让人联想到ContextMenu，然而它和ContextMenu并没有什么关系，它是在Android6.0(API 23)才添加的一个选项，是用于检测外部设备上的按钮是否按下的，例如蓝牙触控笔上的按钮，一般情况下，忽略即可。 |
| [OnDoubleTapListener](https://developer.android.com/reference/android/view/GestureDetector.OnDoubleTapListener.html) | 双击事件，有三个回调类型：双击(DoubleTap)、单击确认(SingleTapConfirmed) 和 双击事件回调(DoubleTapEvent) |
| [OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) | 手势检测，主要有以下类型事件：按下(Down)、 一扔(Fling)、长按(LongPress)、滚动(Scroll)、触摸反馈(ShowPress) 和 单击抬起(SingleTapUp) |
| [SimpleOnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) | 这个是上述三个接口的空实现，一般情况下使用这个比较多，也比较方便。 |

#### OnContextClickListener

由于 OnContextClickListener 主要适用于监测外部设备按钮，所以如果使用它的话，如果监听 onContextCLick(MotionEvent)，则必须在 View 的 onGenericMotionEvent(MotionEvent) 中调用 GestureDetector 的 OnGenericeMotioNEvent(MotionEvent).实际开发中，我们用到这个监听器的场景并不是很多，所以我们重点关注后面几个监听器



#### OnDoubleTapLisner

**这个是用来检测双击事件，它有三个回调接口，分别是 onDoubleTap,onDoubleTapEvent,onSingTapConfirmed**

为什么监听单击事件桶 onSingleTapConfirmed,而不使用 onClickListener?