# Android-View工作原理分析

> 注： **以下相关资料均来自 Android艺术探索，部分内容加入了一些我个人的理解。**

## ViewRoot和DecorView相关分析

ViewRoot对应于 ViewRootImpl 类，它是连接 WindowManager和 DecorView 的纽带，View 的三大流程均是通过ViewRoot来完成的。在ActivityThread 中，当Activity 对象被创建完毕后，会将DecorView添加到Window中，同时会创建 ViewRootImpl 对象，并将ViewRootImpl对象和 DecorView建立关联。

如下源码：**WindowManagerGlobal**

```java
root = new ViewRootImpl(view.getContext(), display);
root.setView(view, wparams, panelParentView);
```



View的绘制流程是从 ViewRoot 的performTraversals方法开始的，它经过measure,layout，和draw三个过程才能最终将一个View绘制出来，其中 measure 用来测量View的宽和高，layout用来确定View在父容器中的放置位置，而draw则负责将View绘制在屏幕上，针对performTraversals的 大致流程，如下图表示：

![image-20200726203753791](https://tva1.sinaimg.cn/large/007S8ZIlly1gh4nmggpquj30qs0j0jsm.jpg)

> 如图所示，performTraversals 会依次调用 performMeasure，performLayout,performDraw这三个方法，这个三个方法分别完成顶级View 的measure,layout和 draw这三大流程，**其中在 performMeasure 中会调用 measure方法，在measure方法中又会调用 onMeasure方法，在 onMeasure 方法中则会对所有的子元素进行 measure过程，这个时候 measure 流程就从父容器传递到子元素中了，这样也就完成了一次 measure过程。接着子元素会重复 父容器的 measure过程，如此反复就完成了整个 View树的遍历。**
>
> 同理，performLayout 和 performDraw 的传递流程和 performMeasure 是类似的，唯一不同的是，performDraw 的传递过程是在 draw 方法中通过 dispatchDraw 来实现的，不过区别并不是很大。

### Measure

measure过程决定了View的 宽/高，Measure完成以后，可以通过 getMeasuredWidth 和 **getMeasuredHeight** 方法来获取到View 测量后的宽/高，在几乎所有的情况下，它都等同于 View的最终 宽/高，(也可以重写 **onSizeChanged**),当然特殊情况除外。

### Layout

Layout过程决定了 getLeft 和 getRight 来拿到View的四个顶点的位置，并可以通过 getWidth 和 getHeight 方法来拿到View 的最终宽/高。

### Draw

Draw过程则决定了 View的显示，只有 draw方法完成以后,View的内容才能显示到屏幕上，但需要注意的是，应该尽量避免在 onDraw的过程中执行过多的循环操作。



### 另一个角度来看DecorView

![image-20200726210548823](https://tva1.sinaimg.cn/large/007S8ZIlly1gh4ofhxogjj30ha0ga0t0.jpg)

DecorView 作为顶级的View,一般情况下 内部会包含一个 竖直方向的 LinearLayout,在这个 LinearLayout 里面有上下两个部分(具体的情况和Android版本及主题有关)，上面是标题栏，下面是内容栏,在Activity 中我们通过 **setContentView** 所设置的布局文件其实就是被加到内容栏之中的，而内容栏的 id 是 content,因此可以理解为 Activity 指定布局的方法不叫 setView 而交 setContentView,因为我们的布局的确加到了 id 为 content 的 FrameLayout中。

#### 获取content -ViewGroup

```kotlin
val content = findViewById<ViewGroup>(android.R.id.content)
```

#### 获取setContentView中设置的View

```kotlin
content.getChildAt(0)
```

最后，DecorView其实是一个 FrameLayout,View层的事件都先经过 DecorView,然后才传递给我们的View.





## MeasureSpec

为了理解View的测量过程，我们不难会去理解 MeasureSpec,从名字上来看，MeasureSpec看起来像测量规格，看起来或多或少的决定了 View 的测量过程。

MeasureSpec很大程度上决定了一个View的尺寸规格，之所以说是很大程度上是因为这个过程受父容器的影响，因为父容器影响 View 的 MeasureSpec 的创建过程。在测量过程中，系统会将 View的 **LayoutParams**根据父容器所施加的规则转换成对应的 MeasureSpec，然后再根据这个 measureSpec 来测量出View的宽高。不过需要注意的是，**这里的宽/高是测量的 宽/高，不一定等于 View 的最终 宽高。**

### 理解MeasureSpec

MeasureSpec 代表一个 32位int 值，高2位代表 SpecMode,低30位代表 SpecSize, SpecMode 是指测量模式，而SpecSize是指在某种测量模式下的规格大小。

**#MeasureSpec**

```kotlin
public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        public static final int AT_MOST     = 2 << MODE_SHIFT;
  
        public static int makeMeasureSpec(@IntRange(from = 0, to = (1 <<MeasureSpec.MODE_SHIFT) - 1) int size,@MeasureSpecMode int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        @MeasureSpecMode
        public static int getMode(int measureSpec) {
            //noinspection ResourceType
            return (measureSpec & MODE_MASK);
        }

        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }
```

MeasureSpec 通过将 **SpecMode** 和 **SpecSize** 打包成一个 **int** 值来避免过多的对象内存分配，为了便于操作，并提供了相应的打包和解包方法。SpecMode 和 SpecSize也是一个int值，一组 SpecMode 和 SpecSize 可以打包为一个 MeasureSpec, 而一个 MeasureSpec 可以通过解包的形式来得出其原始的 SpecMode 和 SpecSize。

**SpecMode* * 有三类，每一类都表示了特殊的含义，如下：

#### UNSPECIFIED

父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态。

#### EXACTLY

父容器已经检测出View所需要的 精确大小，这个时候View的最终大小就是 SpecSize 所指定的值。它对应于 LayoutParams 中的 match_parent 和具体的数值这两种模式。

#### AT_MOST

父容器指定了一个可用大小即 SpecSize, View 的大小不能大于这个值，具体是什么值要看不同 View 的具体实现。它对应 LayoutParams 中的 wrap_content.



### MeaureSpec 和 LayoutParams 的对应关系

系统内部通过 MeasureSpec 来进行View的测量，正常情况下我们使用 View 指定MeasureSpec,尽管如此，但是我们可以给 View 设置LayoutParams，在View测量的时候，系统会将LayoutParams 在父容器的约束下转换成对应的 MasureSpec,然后再根据 这个 MeasureSpec 来确定View测量后的 宽/高。但需要注意的是，MeasureSpec 不是唯一由 LayoutParams 决定的，LayoutParams 需要和父容器一起才能决定View 的 MeasureSpec,从而进一步决定View的 宽/高。

对于顶级View(即DecorView)和普通View来说，MeasureSpec 的转换过程略有不同。对于 DecorView ，其MeasureSpec由窗口的尺寸和其自身的 LayoutParams 来共同决定，MeasureSpec 一旦确定后，onMeasure中就可以确定 View的 测量 宽/高。



**#getRootMeasureSpec**

```java
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }
```

上述代码表示，DecorView 的 MeasureSpec 的产生过程如下，具体来说其遵守以下规则，根据它1的 LayoutParams 中的 宽/高 的参数与来划分。

- LayoutParams.MATCH_PARENT :   精确模式，大小就是窗口的大小；
- LayoutParams.WRAP_CONTENT:  最大模式，大小不定，但是不能超过窗口的大小；
- 固定大小： 精确模式，大小为 LayoutParams 中指定的大小。

对于一个普通View.其 measure过程由ViewGroup 传递而来，我们看一下 ViewGroup 的 measureChildWithMargins 方法。

**#measureChildWithMargins**

```kotlin
  protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
				
    		//子view的measureSpec和view的 margin,padding,layoutParams及父容器 measuureSpec有关
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
				//找出一个view具体应该多大
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

上面的方法会对子view进行测量，在调用子元素的 measure方法之前 会先通过 **getChildMeasureSpec** 方法得到子元素的 **MeasureSpec** ,观察具体的代码，子元素的 MeasureSpec 创建和父容器的 MeasureSpec 和 子元素本身的 LayoutParams 有关，此外还与 View 的 margin 及 Padding有关。具体点代码如下：

**#getChildMeasureSpec**

```java
 public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;
				
   			//先判断父容器测量模式
        switch (specMode) {
        case MeasureSpec.EXACTLY:
            //如果父布局测量模式为exactly并且子view-width>0
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                 //如果父布局测量模式为exactly并且子view-width为match
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                //如果父布局测量模式为exactly并且子view-width为wrap,
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                //父布局为AT_MOST,子view-width>=0.其测量模式为EXACTLY（match或resultSize）
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
               	//父布局为AT_MOST.子view即使match,其测量模式依然AT_MOST(即wrap)
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
               		//父布局为AT_MOST.子view为wrap,其测量模式为AT_MOST(即wrap)
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                //父布局为UNSPCIFIED,子view-width>=0,其测量模式为exactly(match或resultSize)
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
               //父布局为UNSPCIFIED,子view-width为match,其测量模式为UNSPECIFIED（即match）
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                //父布局为UNSPCIFIED,子view-width为wrap,其测量模式为UNSPECIFIED（即match）
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

上述方法，主要作用是根据父容器的 **MasureSpec** 同时结合 View 本身的 LayoutParams 来确定子元素的 MeasureSpec,参数中的 padding是指 父容器已占用的空间大小，因此子元素实际可用大小为父容器的尺寸-padding。如下图及代码

```java
int specSize = MeasureSpec.getSize(spec);
int size = Math.max(0, specSize - padding);
```

![image-20200804214409673](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghf448nxlcj30bl08gdfx.jpg)





对于上述的关系，我们可以整理出如下表来表示：

> **parentSprcMode** 代表父容器测量模式
>
> **childLayoutParams** 代表子view的 layoutParams

![image-20200804215132937](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghf4bw5t4gj30ku06djss.jpg) 



#### 总结

对于普通View，其MeasureSpec 由 父容器 的 MeasureSpec 和 自身的 LayouutParams 共同决定，针对不同的父容器和 View本身不同的 LayoutParams ，View 就可以有多种 MeasureSpec。

当View 采用固定宽高的时候，不管父容器的 MeasureSpec 是什么，view 的 MeasureSpec  都是精确模式 并且其大小遵循 layoutoutparams 中的大小。当view 的宽/高 是 match_parent 时，如果父容器的模式是精准模式，那么View 也是精准模式并且其大小是父容器 的剩余空间；如果父容器的模式是精准模式，那么View 也是精准模式，并且其大小是父容器的剩余空间；如果父容器 是最大 模式，那么 View 也是 最大模式 并且其大小 不会超过 父容器的剩余空间。当View 的宽高是 wrap_content 时，不管父容器 的模式是精准还是最大化，View 的模式 总是最大化并且大小不能超过父容器的剩余空间。

