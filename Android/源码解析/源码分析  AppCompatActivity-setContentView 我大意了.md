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
> - 混元门代码 第三代大弟子，**打工牛子** 参见



## 熟悉的味

![image-20201121144840056](https://tva1.sinaimg.cn/large/0081Kckwly1gkwsngj2g5j31f60bhdto.jpg)

为什么会这样，明明是一个普通的TextView,为什么变成了MaterialTextView，难不成你在逗我。

![马保国语录全文马保国语录原版分享-758手游网](https://tva1.sinaimg.cn/large/0081Kckwly1gkxt77vrk8j306h060748.jpg)



## 顺藤摸瓜

打工人,打工魂,我乃混元门...

呸,跑题了，我们进入正轨，今天非要扒了你的裤衩子。



先进入 `AppCompatActivity` 的 **setContenView()**:

```java
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```

索然无味，进入 `AppCompatDelegateImpl`-**setContentView()**

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

这里就很直接嘛，看过 `Activity-setContentView` 的同学都知道 `contentParent` 是啥，我们的根布局嘛，这里后面的 **inflate** 就不用说了，主要在于 **ensureSubDecor()**，看看它里面做了甚。

---



进入 **ensureSubDecor()**

```java
private void ensureSubDecor() {
    if (!mSubDecorInstalled) {
        mSubDecor = createSubDecor();
      	...省略一大段代码
    }
}
```

`mSubDecorInstalled` 见名之意，`windows` 是否已经安装了 `DecorView`,如果已经安装，就忽略。

我为什么会知道呢？翻译啊,ohhhh。

```java
// true if we have installed a window sub-decor layout.
private boolean mSubDecorInstalled;
```



---

进入 **createSubDecor()**

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

这个方法相对较复杂，理解了其，这篇也就可以结束了。

1. `TypedArray`，是不是有种似曾相识的感觉，没错，这里和 **Activity-setContentView** 源码里的一样，也都是判断主题，从而决定一些配置及最终的根布局文件;

2. 初始化 **AppCompatDategateImpl** 的 `mWinodws`;

   ```java
   private void ensureWindow() {
       if (mWindow == null && mHost instanceof Activity) {
           attachToWindow(((Activity) mHost).getWindow());
       }
   }
   ```

3. **getDecorView()**，其内部即 **PhoneWindows.getDecorView()** 方法

   ```java
   @Override
   public final @NonNull View getDecorView() {
       if (mDecor == null || mForceDecorInstall) {
           installDecor();
       }
       return mDecor;
   }
   ```

   **installDecor()** 方法上个博文提过了，就是用它确保我们的 `DecorView` 与 `mContentParent` 已经初始化。

4. 这里获取了一个 ***R.layout.abc_screen_simple*** 的布局，我们可以理解为，这是 **AppCompatActivity** 特定的根布局。

5. 这里获取了 **ContentFrameLayout** 与DecorView的 `content` ，获取的意义是什么呢？到了下一步就知道；

6. 遍历 **windowsContent**，将其中的所有 **view** 添加到 `contentView` 中，并移除windowsContent中相应的view.最后，再设置 contentView 其`id`为R.id.content(原本为***action_bar_activity_content***)，将**windowsContent** `id` 置 **null**.

7. 最后将新的 `contentView` 设置到 `windows`,即也就是将 `contentView` **add**到我们的根容器 `contenParent` 里，内部是如下方法。

   ```java
   @Override
   public void setContentView(View view, ViewGroup.LayoutParams params) 
       	  ...
           mContentParent.removeAllViews();
           mContentParent.addView(view, params);
   				...
   }
   ```

   先清空根布局，再add 刚才传入的布局，因为我们在第 6 步更改过id,所以相当于这个新的容器即为我们新的 `根ViewGroup`.

### 阶段小思考

问题来了，为什么源码里要将 **PhoneWindows**-`windowsContent` 里的所有view复制到新的 `contentView` 里？

> 因为，Activity 默认的 `DecorView` 加载的是***R.layout.screen_simple***,而我们的根布局是其中的一个`子FrameLayout`,当使用 **AppCompatActivity** 时，为了兼容性，其有自己相应的主题layout,所以在设置时，先将当前根容器里的所有子view放到新的这个容器里，再将这个容器的id设置为**R.id.content**，即让其成为新的根容器，再将这个容器add到我们的 `DecorView` 里的根布局(即FrameLayout)上,这样就达到了在不影响原有view显示情况下的兼容效果。

这样说你明白了吗？

![微信朋友圈能评论表情包了，来斗图啊！_手机新浪网](https://tva1.sinaimg.cn/large/0081Kckwly1gkxsz9qtk6j306o06owei.jpg)

还不明白？好吧，我再从 `背景` 说一遍

> 正如上面所述，**AppCompatActivity** 有自己特定的容器 layout,如果在设计时，让它直接替代了Activity的默认根容器，就意味着 **AppCompatActivity** 必须独立的去写一份，这合适吗，显然不适合。所以正因为如此，**AppCompatActivity** 里的 **DecorView** 变量名叫做 `mSubDecor`,而我们基础 **PhoneWindows** 里的叫做 `mDecor`，再想想为什么**AppCompatActivity** 里会单独再定义一个 所谓的 **DecorView**，意义何在，再配合**AppCompatDelegateImpl-createSubDecor()**方法里上述的操作，现在你懂了吗？

具体如下图所示：

![image-20201122111751708](https://tva1.sinaimg.cn/large/0081Kckwly1gkxse09m7tj30zc0me769.jpg)

检查层级我们就会发现，原来AppCompatActivity 是在原 Activity 布局层级上嵌套的，正如上面所描述，是不是有种ohhhh，就这啊的感觉。

![你品你细品表情包_抖音你品你细品高清表情包下载_游戏吧](https://tva1.sinaimg.cn/large/0081Kckwly1gkxsj9u4itj307r051a9y.jpg)

### 串一下思路

1. 当我们在 `AppCompatActivity` 里调用 **setContentView()** 时，其内部调用的是`AppCompatDelegateImpl`的 **setContentView()**,最终调用了**ensureSubDecor()**,即用来确保DecorView是否已经初始化成功。
2. 在 **ensureSubDecor()** 方法里，先判断 `子DecorView`(为什么是`子`,因为它不可能直接替代根DecorView,AppCompatActivity只是做了一个兼容，即在`DecorView`之上再添加一个副级，从而做到**对调用者屏蔽，对于使用者而言，其实毫无察觉**) 有没有安装，如果没有，则调用 **createSubDecor()**去初始化它;
3. **createSubDecor()** 内部会根据当前主题进行相关配置，最终设置当前的根容器，并将当前 `Windows-DecorView` 根容器-FrameLayout里的所有子view全部add到新的容器里，再将新容器的id改为 ***R.id.content***,然后**windows.setContentView()**，内部即 ***add*** 到旧容器 `FrameLayou`上，成为唯一子容器。而这个新的容器就是最新的根容器。
4. 随后的方法很简单，我们自己的布局直接 ***add*** 到根容器 ViewGroup上即可。







### 为什么打印不一致

等等，最开始的 `View  `打印增加前缀的原因是啥，这和 **setContentView()** 有何关系？

![搞笑端碗等饭表情包大全抖音端碗等饭表情包_抖音_图文教程_多特软件站](https://tva1.sinaimg.cn/large/0081Kckwly1gkxskg5psfj307705fmx0.jpg)

> 是啊，好像没什么关系，那你在这说xxx,不好意思，我们切换下一个话题。

我们回到最开始的 ***AppCompatActivity-Create()***

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

没有什么说的，我们直接进入 **installViewFactory()**

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

如果我们没有设置 `LayoutInflater` 工厂，则会设置默认的工厂，然后最终创建布局时会调用 **onCreateView()** 方法。

---

我们进入相应的 ***onCreateView()***

```java
@Override
public View createView(View parent, final String name, @NonNull Context context,
        @NonNull AttributeSet attrs) {
  	...
    return mAppCompatViewInflater.createView(parent, name, context, attrs, inheritContext,IS_PRE_LOLLIPOP,true,VectorEnabledTintResources.shouldBeUsed()
    );
}
```

没什么好说的，进入 ***mAppCompatViewInflater.createView()***

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

哦呵呵呵，原来这里是对我们的默认的 `View` 进行了替换，这也就是为什么我们使用`AppCompatActivity` 打印出来的子 `View` 自带了前缀显示。











