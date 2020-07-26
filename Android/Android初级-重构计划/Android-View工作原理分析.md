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