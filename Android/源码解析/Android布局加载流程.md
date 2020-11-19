# 从 setContenView源码解析

## 起因

- 你知道 一个 xml布局如何加载成最终的布局吗？
- 你知道 我们常用的 setContentView中做了什么？
- 你知道 LayoutInflater 内部做了什么？

## Activity-setContentView

### 具体框架图如下

![image-20201119184615634](https://tva1.sinaimg.cn/large/0081Kckwly1gkuoa1ojtqj30rc0g5gol.jpg)

### Activity

#### - setContentView()

```java
public void setContentView(@LayoutRes int layoutResID) {
  			//windows只有一个具体实现类，即 PhoneWindows
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
}
```

####  -- getWindow()

```java
public Window getWindow() {
    return mWindow;
}
```

#### 什么是Windows?

Windows 表示一个窗口的概念，Android 中不管是Activity,dialog,还是 Toast 它们的视图都是附加在 Windows 上，因此可以称 windows 是View的直接管理者。而Windows也只有一个子类，即PhoneWindows.



### PhoneWindow

#### setContentView()

```java
@Override
public void setContentView(int layoutResID) {
  	//判断父容器(DecorView)是否为null,否则进行系统初始化工作
    if (mContentParent == null) {
        installDecor(); //1
    }
  	...
    //将传入的布局加载到 contentParent上-> inflate
    mLayoutInflater.inflate(layoutResID, mContentParent);
    ...
}
```

#### - installDecor()

```java
private void installDecor() {
    mForceDecorInstall = false;
    if (mDecor == null) {
      	//可以理解为初始化DecorView
        mDecor = generateDecor(-1); //1
        ...
    } else {
        mDecor.setWindow(this);
    }
    if (mContentParent == null) {
       //解析系统资源layout文件，将其加载到 DecorView,即加载 R.layout.screen_simple
       mContentParent = generateLayout(mDecor); //2
  ...
}
```

#### -- geteratDecor()

```java
//生成DecorView
protected DecorView generateDecor(int featureId) {
    ...
    return new DecorView(context, featureId, this, getAttributes());
}
```

### -- generateLayout()

```java
//加载不同的布局给 layoutResource
protected ViewGroup generateLayout(DecorView decor) {
  			//获取当前主题资源
  			TypedArray a = getWindowStyle();
        int layoutResource;
        // 根据当前主题判断，判断 layoutResourtce到底使用哪个根布局文件
        if(){}else if(){}else if(){
            // 默认的主布局
            layoutResource = R.layout.screen_simple;
        }
        
        mDecor.startChanging();
        //将布局加载到 mDecor
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

 				//获取DecorView-R.id.content对应的ViewGroup(是一个FrameLayout)
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        
        // 返回获取的viewGroup
        return contentParent;
}
```

---



## AppCompatActivity-setContentView

### 整体流程

- 当我们的父类是 AppCompatActivity时，调用setContentView时，其内部通过获取windows,初始化一个DecorView,并手动设置contentId.

### AppCompatActivity

#### setContentView

```java
@Override
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```

### AppCompatDelegateImpl

#### setContentView()

```java
   @Override
    public void setContentView(int resId) {
        ensureSubDecor();
      	//直接获取根布局content,然后添加指定的布局到content中 
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mAppCompatWindowCallback.getWrapped().onContentChanged();
    }
```

#### ensureSubDecor()

```java
private void ensureSubDecor() {
    if (!mSubDecorInstalled) {
        mSubDecor = createSubDecor();
     ...
    }
}
```

### createSubDecor()

```java
 private ViewGroup createSubDecor() {
   			//获取主题
        TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);
				...
        //确定windows实例化
        ensureWindow(); //1
        mWindow.getDecorView();
   			...
        //开始解析DecorView
         subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
        ...
        //获取decorview里的content(即ContentFrameLayout)
       final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);
        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            ...
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);
            ...
        }
        mWindow.setContentView(subDecor);
   		  return subDecor;
 }

//1-> 初始化windows
private void ensureWindow() {
    // We lazily fetch the Window for Activities, to allow DayNight to apply in
    // attachBaseContext
    if (mWindow == null && mHost instanceof Activity) {
        attachToWindow(((Activity) mHost).getWindow());
    }
    if (mWindow == null) {
        throw new IllegalStateException("We have not been given a Window");
    }
}
```

#### installViewFactory()

```java
@Override
public void installViewFactory() {
    LayoutInflater layoutInflater = LayoutInflater.from(mContext);
    if (layoutInflater.getFactory() == null) {
      	//设置了Factory,也即创建view时就会走自己的onCreateView方法
        LayoutInflaterCompat.setFactory2(layoutInflater, this);
    } else {
        if (!(layoutInflater.getFactory2() instanceof AppCompatDelegateImpl)) {
            Log.i(TAG, "The Activity's LayoutInflater already has a Factory installed"
                    + " so we can not install AppCompat's");
        }
    }
}
```

#### createView()

```java
final View createView(View parent, final String name, @NonNull Context context,
        @NonNull AttributeSet attrs, boolean inheritContext,
        boolean readAndroidTheme, boolean readAppTheme, boolean wrapContext) {
  	...
    View view = null;
		//对view进行替换
    switch (name) {
        case "TextView":
            view = createTextView(context, attrs);
            verifyNotNull(view, name);
            break;
        case "ImageView":
            view = createImageView(context, attrs);
            verifyNotNull(view, name);
            break;
    ...
   if (view == null && originalContext != context) {
     	//自定义view时
       view = createViewFromTag(context, name, attrs); //注意点
   }
   ...
    return view;
 }
```



### AppCompatViewInflater

#### createViewFromTag

```java
private View createViewFromTag(Context context, String name, AttributeSet attrs) {
	...
  return createViewByPrefix(context, name, null);
}
```

#### createViewByPrefix

```java
private View createViewByPrefix(Context context, String name, String prefix)
        throws ClassNotFoundException, InflateException {
  	//从缓存中获取
    Constructor<? extends View> constructor = sConstructorMap.get(name);
    try {
        if (constructor == null) {
            Class<? extends View> clazz = Class.forName(
                    prefix != null ? (prefix + name) : name,
                    false,
                    context.getClassLoader()).asSubclass(View.class);
						//反射创建构造函数
            constructor = clazz.getConstructor(sConstructorSignature);
            sConstructorMap.put(name, constructor);
        }
        constructor.setAccessible(true);
      	//反射创建view实例
        return constructor.newInstance(mConstructorArgs);
    } catch (Exception e) {
        return null;
    }
}
```







