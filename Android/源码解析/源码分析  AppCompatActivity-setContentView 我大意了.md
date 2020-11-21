# 源码分析 | AppCompatActivity-setContentView 我大意了

![耗子尾汁，云开发的羊毛人人可薅！ - w3cschool编程狮| 微信公众号文章阅读- WeMP](https://tva1.sinaimg.cn/large/0081Kckwly1gkwrv0y4wrj309k042q2s.jpg)

## HZWZ

现在的年轻人一上来就粘源码，对我这样一个小菜瓜，这样合适吗，这样不合适。

## 背景

故事是这样开始的：

> - 有一天，我发现自己写的布局没有
> - 按照我的想法打印
> - 带上了莫名其妙的开头
>
> 
>
> - 有一天，两个年轻人，不讲武德
>
> - 非要告诉我这是AppCompatActivity的原因
>
> - 我不信
>
> - 他们偷袭，显然是有备而来
>
>   
>
> - 我大意了
> - 我没有闪
> - 今天，我要自证事实
> - 混元门代码 第三代大弟子，打工牛子 参见



## 熟悉的味

![image-20201121144840056](https://tva1.sinaimg.cn/large/0081Kckwly1gkwsngj2g5j31f60bhdto.jpg)

为什么会这样，明明是一个普通的TextView,为什么变成了MaterialTextView，难不成你在逗我。

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkwsz6uoolj31e10s5jsy.jpg" alt="img" style="zoom:33%;" />



## 顺藤摸瓜

打工人,打工魂,我乃混元门...

呸,跑题了，我们进入正轨，今天非要扒了你的裤衩子。



先进入AppCompatActivity 的 setContenView():

```java
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```

索然无味，进入 AppCompatDelegateImpl-setContentView()

```java
@Override
public void setContentView(int resId) {
  	//1
    ensureSubDecor();
    ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
    contentParent.removeAllViews();
    LayoutInflater.from(mContext).inflate(resId, contentParent);
    mAppCompatWindowCallback.getWrapped().onContentChanged();
}
```

这里就很直接嘛，看过Activity-setContentView的同学都知道 contentParent 是啥，我们的根布局嘛，这里后面的 inflate就不用说了，主要在于 ensureSubDecor()，看看它里面做了甚。

---



进入 ensureSubDecor()

```java
private void ensureSubDecor() {
    if (!mSubDecorInstalled) {
        mSubDecor = createSubDecor();
      	...省略一大段代码
    }
}
```

mSubDecorInstalled 见名之意，windows是否已经安装了DecorView,如果已经安装，就忽略。

我为什么会知道呢？翻译啊。

```java
// true if we have installed a window sub-decor layout.
private boolean mSubDecorInstalled;
```



---

进入createSubDecor()

```java
    private ViewGroup createSubDecor() {
      	//1
        TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);
      	if(xxx)xxx
      	//2
        ensureWindow();
      	//3
        mWindow.getDecorView();
				...//忽略一段代码
        ViewGroup subDecor = null;
      
      	//4 
         subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
				//5
        final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);
        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
      	
      	//6
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);
        }

        //7
        mWindow.setContentView(subDecor);
        return subDecor;
    }

```

这个方法相对较复杂，理解了其，这篇也就可以结束了。哦呵呵呵....

1. TypedArray，是不是有种似曾相识的感觉，没问题，这里和Activity-setContentView源码里的一样，也都是判断主题，从而决定一些配置及最终的根布局文件

2. 初始化AppCompatDategateImpl的 mWinodws

   ```java
   private void ensureWindow() {
       if (mWindow == null && mHost instanceof Activity) {
           attachToWindow(((Activity) mHost).getWindow());
       }
   }
   ```

3. getDecorView，其内部即PhoneWindows.getDecorView()方法

   ```java
   @Override
   public final @NonNull View getDecorView() {
       if (mDecor == null || mForceDecorInstall) {
           installDecor();
       }
       return mDecor;
   }
   ```

   installDecor() 方法上个博文提过了，就是用它确保我们的DecorView与 mContentParent已经安装成功。

4. 这里获取了一个 R.layout.abc_screen_simple 的布局，我们可以理解为，这是AppCompatActivity 特定的根布局。

   ```xml
   <androidx.appcompat.widget.FitWindowsLinearLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       android:id="@+id/action_bar_root"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:orientation="vertical"
       android:fitsSystemWindows="true">
   
       <androidx.appcompat.widget.ViewStubCompat
           android:id="@+id/action_mode_bar_stub"
           android:inflatedId="@+id/action_mode_bar"
           android:layout="@layout/abc_action_mode_bar"
           android:layout_width="match_parent"
           android:layout_height="wrap_content" />
   
       <include layout="@layout/abc_screen_content_include" />
   
   </androidx.appcompat.widget.FitWindowsLinearLayout>
   ```

   **abc_screen_content_include**

   ```xml
   <merge xmlns:android="http://schemas.android.com/apk/res/android">
   
       <androidx.appcompat.widget.ContentFrameLayout
               android:id="@id/action_bar_activity_content"
               android:layout_width="match_parent"
               android:layout_height="match_parent"
               android:foregroundGravity="fill_horizontal|top"
               android:foreground="?android:attr/windowContentOverlay" />
   
   </merge>
   ```

5. 这里获取了ContentFrameLayout与DecorView的content，获取的意义是什么呢？到了下一步就知道；

6. 遍历 windowsContent，将其中的所有view 添加到 contentView 中，并移除windowsContent中相应的view.最后，再设置contentView 其id为R.id.content(原本为action_bar_activity_content)，将windowsContent Id置null.

7. 最后将新的 contentView 设置到windows,即也就是将contentView add到我们的根布局contenParent里，内部是如下方法。

   ```java
   @Override
   public void setContentView(View view, ViewGroup.LayoutParams params) 
       	  ...
           mContentParent.removeAllViews();
           mContentParent.addView(view, params);
   				...
   }
   ```

   先清空根布局，再add 刚才传入的布局，因为我们在第 6 步更改过id,所以相当于

问题来了，为什么源码里要将PhoneWindows-windowsContent里的所有view复制到新的 contentView里？

> 因为，Activity默认的DecorView加载的是R.layout.screen_simple,而我们的根布局是其中的一个子FrameLayout,当使用 AppCompatActivity时，为了兼容性，其有自己相应的主题layout,所以在设置时，先将当前根View 里的所有子view放到新的这个容器里，再将这个容器的id设置为R.id.content，即让其成为新的根容器，再将这个容器add到我们的 DecorView 里的根布局(即FraeLayout)上,这样就达到了在不影响原有view显示情况下的兼容效果。

如下图所示：

![image-20201121174012239](https://tva1.sinaimg.cn/large/0081Kckwly1gkwxlxk87yj311p0kj0vi.jpg)

检查层级我们就会发现，原来AppCompatActivity 是在原 Activity 布局层级上嵌套的，正如上面所描述。



问题又来了，最开始的TextView增加后缀的原因是啥？

我们回到最开始的AppCompatActivity-Create()

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    final AppCompatDelegate delegate = getDelegate();
  	//入口
    delegate.installViewFactory();
    delegate.onCreate(savedInstanceState);
    super.onCreate(savedInstanceState);
}
```

没有什么说的，我们直接进入 installViewFactory()

```java
@Override
public void installViewFactory() {
    LayoutInflater layoutInflater = LayoutInflater.from(mContext);
  	//1
    if (layoutInflater.getFactory() == null) {
        LayoutInflaterCompat.setFactory2(layoutInflater, this);
    }
  	...
}
```

如果我们没有设置LayoutInflater工厂，则会设置默认的工厂，然后最终创建布局时会调用 onCreateView 方法。



我们进入相应的onCreateView

```java
@Override
public View createView(View parent, final String name, @NonNull Context context,
        @NonNull AttributeSet attrs) {
  	...
    return mAppCompatViewInflater.createView(parent, name, context, attrs, inheritContext,IS_PRE_LOLLIPOP,true,VectorEnabledTintResources.shouldBeUsed()
    );
}
```

```java
final View createView(View parent, final String name, @NonNull Context context,
        @NonNull AttributeSet attrs, boolean inheritContext,
        boolean readAndroidTheme, boolean readAppTheme, boolean wrapContext) {
		...
    switch (name) {
        case "TextView":
            view = createTextView(context, attrs);
            verifyNotNull(view, name);
            break;
        ...
```

相应的，createView 里面对我们的默认的View进行了替换，这也就是为什么我们使用AppCompatActivity 打印出来的子View自带了前缀显示。

