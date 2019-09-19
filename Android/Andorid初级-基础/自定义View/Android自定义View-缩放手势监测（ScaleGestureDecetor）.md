# Android自定义View-缩放手势监测（ScaleGestureDecetor）

***注明:  非常感谢 [gcssloop](www.gcssloop.com) 的博客，以下为我学习时的笔记记录。***

> 学习本文前，建议先学习 自定义View-手势检测(GestureDetector)及拥有一定的事件分发基础。



缩放手势监测（ScaleGestureDecetor）是官方提供的一个手势监测工具，它的使用方式与 GentureDetector 类似，也是通过 Listener 进行监听用户的操作手势，它是对缩放手势进行了一次封装，可以方便用户快速的完成缩放功能的开发。下面我们就来看它是怎么实现的吧。

## 构造方法

它有两个构造方法，和 GestureDetector 类似。

```
ScaleGestureDetector(Context context, ScaleGestureDetector.OnScaleGestureListener listener)

ScaleGestureDetector(Context context, ScaleGestureDetector.OnScaleGestureListener listener, Handler handler)
```



## 手势监听器

它有两个监听器，但严格来说，这两个简体器是同一个，只不过一个是接口，另一个是空实现。

| 监听器                       | 简介                     |
| ---------------------------- | ------------------------ |
| OnScaleGestureListener       | 缩放手势检测器。         |
| SimpleOnScaleGestureListener | 缩放手势检测器的空实现。 |



### OnScaleGestureListener

**缩放手势监听器有 3 个方法：**

| 方法                                                         | 简介                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| boolean [onScaleBegin](https://developer.android.com/reference/android/view/ScaleGestureDetector.OnScaleGestureListener.html#onScaleBegin(android.view.ScaleGestureDetector))([ScaleGestureDetector](https://developer.android.com/reference/android/view/ScaleGestureDetector.html)detector) | 缩放手势开始，当两个手指放在屏幕上的时候会调用该方法(只调用一次)。如果返回 false 则表示不使用当前这次缩放手势。 |
| boolean [onScale](https://developer.android.com/reference/android/view/ScaleGestureDetector.OnScaleGestureListener.html#onScale(android.view.ScaleGestureDetector))([ScaleGestureDetector](https://developer.android.com/reference/android/view/ScaleGestureDetector.html)detector) | 缩放被触发(会调用0次或者多次)，如果返回 true 则表示当前缩放事件已经被处理，检测器会重新积累缩放因子，返回 false 则会继续积累缩放因子。 |
| void [onScaleEnd](https://developer.android.com/reference/android/view/ScaleGestureDetector.OnScaleGestureListener.html#onScaleEnd(android.view.ScaleGestureDetector))([ScaleGestureDetector](https://developer.android.com/reference/android/view/ScaleGestureDetector.html)detector) | 缩放手势结束。                                               |

```java
ScaleGestureDetector scaleGestureDetector = new ScaleGestureDetector(this, new ScaleGestureDetector.OnScaleGestureListener() {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        //缩放中心，x坐标
        Log.e("demo","focusX="+detector.getFocusX());
        //缩放中心，y坐标
        Log.e("demo","focusY="+detector.getFocusY());
        //缩放因子
        Log.e("demo","scale="+detector.getScaleFactor());

        return true;
    }

    @Override
    public boolean onScaleBegin(ScaleGestureDetector detector) {
        return true;
    }

    @Override
    public void onScaleEnd(ScaleGestureDetector detector) {

    }
});
img_happy.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        return scaleGestureDetector.onTouchEvent(event);
    }
});
```



## 基本原理

**在缩放手势中我们其实主要关系的只有两个参数，一个是缩放的中心点，另一个就是缩放比例了**。下面我们来看看这两个参数是如何计算出来的。

### 计算缩放的中心点(焦点)

如果只有两个手指的话，缩放的中心点自然是非常容易计算的，那就是两个手指坐标的中点，但是如果有多个手指该如何计算缩放的中心点呢？

**计算中心点的原理其实也非常简单，那就是将所有的坐标都加起来，然后除以数量即可。**

这个数学原理并并不难，我们随便画一下就能看出来。不过在实际运用中，用户的手指数量可能并不是固定的，用户可能随时抬起来或者按下手指，ScakeGestureDetector 中是这样实现的：

```java
 final boolean anchoredScaleCancelled =
            mAnchoredScaleMode == ANCHORED_SCALE_MODE_STYLUS && !isStylusButtonDown;
    final boolean streamComplete = action == MotionEvent.ACTION_UP ||
            action == MotionEvent.ACTION_CANCEL || anchoredScaleCancelled;

    // 注意这里
    if (action == MotionEvent.ACTION_DOWN || streamComplete) {
		//重置侦听器正在进行的任何缩放。
        //如果是ACTION_DOWN，我们正在开始一个新的事件流。
        //这意味着应用程序可能没有给我们所有的事件(事件被上层直接拦截了)。
        if (mInProgress) {
            mListener.onScaleEnd(this);
            mInProgress = false;
            mInitialSpan = 0;
            mAnchoredScaleMode = ANCHORED_SCALE_MODE_NONE;
        } else if (inAnchoredScaleMode() && streamComplete) {
            mInProgress = false;
            mInitialSpan = 0;
            mAnchoredScaleMode = ANCHORED_SCALE_MODE_NONE;
        }

        if (streamComplete) {
            return true;
        }
    }
```

可以看到，当触发 **down** 或者 触发**up**,**cancel** 时，如果之前处于缩放计算的状态，会将其状态重置，并调用 **onScaleEnd** 方法。mAnchoredScaleMode ，这些是对触控笔等外设的支持，我们一般用不到。

### 计算中心点：

```java
inal boolean configChanged = action == MotionEvent.ACTION_DOWN ||
        action == MotionEvent.ACTION_POINTER_UP ||
        action == MotionEvent.ACTION_POINTER_DOWN || anchoredScaleCancelled;

// 注意这里
final boolean pointerUp = action == MotionEvent.ACTION_POINTER_UP;
final int skipIndex = pointerUp ? event.getActionIndex() : -1;

// 确定焦点
float sumX = 0, sumY = 0;
final int div = pointerUp ? count - 1 : count;
final float focusX;
final float focusY;
if (inAnchoredScaleMode()) {
    // 在锚定比例模式下，焦点始终是双击或按钮按下时手势开始的位置
    focusX = mAnchoredScaleStartX;
    focusY = mAnchoredScaleStartY;
    if (event.getY() < focusY) {
        mEventBeforeOrAboveStartingGestureEvent = true;
    } else {
        mEventBeforeOrAboveStartingGestureEvent = false;
    }
} else {
	// 注意这里， 最终计算得到焦点
    for (int i = 0; i < count; i++) {
        if (skipIndex == i) continue;
        sumX += event.getX(i);
        sumY += event.getY(i);
    }

    focusX = sumX / div;
    focusY = sumY / div;
}
```



### 计算缩放比例

缩放比例计算，就是计算各个手指到焦点的平均距离，在用户手指移动后用新的平均距离除以旧的平均距离，并以此计算得出缩放比例。

```java
// 计算到焦点的平均距离
float devSumX = 0, devSumY = 0;
for (int i = 0; i < count; i++) {
    if (skipIndex == i) continue;
    devSumX += Math.abs(event.getX(i) - focusX);
    devSumY += Math.abs(event.getY(i) - focusY);
}
final float devX = devSumX / div;
final float devY = devSumY / div;

// 注意这里
final float spanX = devX * 2;
final float spanY = devY * 2;
final float span;
if (inAnchoredScaleMode()) {
    span = spanY;
} else {
    // 相当于 sqrt(x*x + y*y)
    span = (float) Math.hypot(spanX, spanY);
}
```

当用户移动的距离超过一定数值(数值的大小由系统定义)后，会触发 onScaleBegin 方法，如果用户在 onScaleBegin 方法里返回了true，表示接受事件后，就会重置缩放相关数值，并且开始积累缩放因子。

```java
// mSpanSlop 和 mMinSpan 都是从系统里面取得的预定义数值，该数值实际上影响的是缩放的灵敏度。
// 不过该参数并没有提供设置的方法，如果对灵敏度不满意的话，和通过直接之际复制一个 ScaleGestureDetector 到项目中， 并且修改其中的数值。
final int minSpan = inAnchoredScaleMode() ? mSpanSlop : mMinSpan;
if (!mInProgress && span >=  minSpan &&
        (wasInProgress || Math.abs(span - mInitialSpan) > mSpanSlop)) {
    mPrevSpanX = mCurrSpanX = spanX;
    mPrevSpanY = mCurrSpanY = spanY;
    mPrevSpan = mCurrSpan = span;
    mPrevTime = mCurrTime;
    mInProgress = mListener.onScaleBegin(this);
}
```

通知用户缩放：

```java
if (action == MotionEvent.ACTION_MOVE) {
    mCurrSpanX = spanX;
    mCurrSpanY = spanY;
    mCurrSpan = span;

    boolean updatePrev = true;

    if (mInProgress) {
        // 注意这里，用户的返回值决定了是否重新计算缩放因子
        updatePrev = mListener.onScale(this);
    }

    // 如果用户返回了 true ，就会重新计算缩放因子
    if (updatePrev) {
        mPrevSpanX = mCurrSpanX;
        mPrevSpanY = mCurrSpanY;
        mPrevSpan = mCurrSpan;
        mPrevTime = mCurrTime;
    }
}
```



**更多Android开发知识请访问—— [Android开发日常笔记](https://github.com/Petterpx/Android_NoteBook)，欢迎Star,你的小小点赞，是对我的莫大鼓励。**