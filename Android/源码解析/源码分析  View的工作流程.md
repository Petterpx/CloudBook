# 源码分析 | View的工作流程

View的工作流程主要是指 measure,layout,draw这三大流程，即测量，布局和绘制，其中 measure 确定View的测量 宽/高，layout确定View的最终宽/高和四个顶点的位置，而draw则将View绘制到屏幕上。

## measure 过程

measure过程要分情况来看，如果只是一个原始的View,那么通过measure方法就完成了测量其测量过程,如果是一个 ViewGroup ,除了完成自己的测量过程外，还会遍历去调用所有子元素的 measure 方法，各个子元素再递归去执行这个流程。

### View的 measure过程

View#onMeasure

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

先看一下 setMeasuredDimension ,其用于设置view 宽/高的测量值，因此我们直接关注其 getDefaultSize 即可。



View#getDefaultSize

```java
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
  	//获取测量模式
    int specMode = MeasureSpec.getMode(measureSpec);
  	//获取测量大小
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```

如源码所示，getDefaultSize 返回的大小就是 measureSpec 中的specSiaze,而这个 specSize正是View 测量后的大小，但是需要注意的是，这里测量后的大小并非最终大小，view的最终大小会在 layout 方法中确定，不过几乎所有情况下 View 的测量大小和最终大小是相等的。

需要注意的是，当specMode是  MeasureSpec.UNSPECIFIED 时，result即为size,也就是等于我们 getDefaultSize中传递的 getSuggestedMinimumWidth()与getSuggestedMinimumHeight(),所以我们去看一下 getSuggestedMinimumWidth的具体实现：



View#getSuggestedMinimumWidth()

```java
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, 			  mBackground.getMinimumWidth());
}
```

如果View没有设置背景，那么View的宽度为 mMinWidth,而 mMinWidth 对应于 android:minWidth这个属性所指定的值，如果这个属性没有指定，那么默认即为0，如果View指定了背景，则View的宽度为 max(mMinWidth, 			  mBackground.getMinimumWidth()) 。其中 mBackground.getMinimumWidth() 是什么呢，我们继续看一下源码,如下所示：

Drawable#getMinimumWidth()

```java
public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}
```

可以看出，getMinimumWidth() 返回的其实是这个Drawable的原始宽度，当然如果这个Drawable没有原始宽度，则返回0。



