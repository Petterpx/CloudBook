# 源码分析 | Activity-setContentView 我都不带闪

我知道大家都很讨厌读别人写的源码分析，因为动不动就长篇大论，不讲武德，这样合适吗，这样不合适。于是，这是一篇不一样的源码分析,如果看完你还说不懂。年轻人，我劝你：

![耗子尾汁是什么梗耗子尾汁意思解析及出处介绍-微侠手游网](https://tva1.sinaimg.cn/large/0081Kckwly1gkwqjnnrz1j309208w74f.jpg)

## 引言

普通的一个 Activity-**setContentView()**，你知道它内部做了什么吗？

## 概要

![image-20201120151757258](https://tva1.sinaimg.cn/large/0081Kckwly1gkwpemuxjjj30re0g477c.jpg)



## 源码分析

我们先来看一下Activity-setContentView方法：

```java
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```

简简单单滴方法，内部调用了 **getWindows.setContentView**(xxx)

等等，`Windows` 是什么？

> `Windows` 表示一个窗口的概念，Android 中不管是Activity,dialog,还是 Toast 它们的视图都是附加在 Windows 上，因此可以称 windows 是View的直接管理者。而Windows也只有实现类，即PhoneWindows.

---

我们接着去看 `PhoneWindows` 的 **setContentView**()

```JAVA
@Override
public void setContentView(int layoutResID) {
    if (mContentParent == null) {
        installDecor(); //关注点
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }
     mLayoutInflater.inflate(layoutResID, mContentParent);
}
```

简简单单，内部执行了一个判断，然后调用 **installDecor() ** 方法。

等等，`mContenParent`是什么？

> mContentParent 即放置我们自己布局的容器，你可以理解为，它是我们的根容器，详情看图。

---

我们接着去看 **installDecor()**方法：

```java
private void installDecor() {
  	...忽略掉
    mDecor = generateDecor(-1); //关注点1
  	...忽略掉
  	if (mContentParent == null) {
      //关注点2
      mContentParent = generateLayout(mDecor);
    }
  	...忽略掉一大段
}
```

这个方法内部很繁琐，很臭很长，我们需要关注这么多吗，不需要，所以直接先进入 **generateDecor()**.

等等，`mDecor` 是什么？

> **mDecor** 是 `windows` 唯一视图，也就是我们 `mContentParent` 的爸爸。简称 **DecorView**,是不是回忆起了点什么。

---

我们接着去看 **generateDecor()** 方法

```java
protected DecorView generateDecor(int featureId) {
  	...忽略一大段
    return new DecorView(context, featureId, this, getAttributes());
}
```

哇，这个方法爱了爱了，直接new了一个 `DecorView`,其他也没啥，返回回去看 **installDecor()** 中的关注点2-**generateLayout()**.

---

我们进入 **generateLayout()** 方法：

```java
protected ViewGroup generateLayout(DecorView decor) {
   //1
   TypedArray a = getWindowStyle();
   if(xx)else if(xxx)	
   else {
       layoutResource = R.layout.screen_simple;
   }
   //2
   mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
	 ..忽略掉一部分
   //3
   ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
   return contentParent;
}
```

1. 显示获取当前主题，然后开始判断究竟要用那个布局
2. 将其加载到 **DecorView** 中
3. 通过 findViewById(内部就是**DecorView.findViewById**)获取 R.id.content，并返回此viewGroup。

等等，这个 `R.layout.screen_simple` 是甚？

![image-20201121131940286](https://tva1.sinaimg.cn/large/0081Kckwly1gkwq2urevoj30mg0eogoj.jpg)

哦，这就是我们DecorView中加载的布局啊，具体大图如下。

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkwxhqrpooj30ut0n60v2.jpg" alt="image-20201121173610246" style="zoom: 67%;" />

如上图所示，我们的布局最终会被添加到这个根布局content中。

## 串一遍思路

我们接下来将上面的分析整体走一遍：

- 当我们调用Activity的 **setContentView** 时，内部其实是执行了 `PhoneWindows`(windows的唯一实例)的 **setContenView()** ；
- 而 `PhoneWindows` 的 **setContentView()** 内部会先判断当前有没有布局容器 `contentParent`，也即就是有没有 `DecorView`，如果没有，执行 **installDecor()** 去初始化我们的 `DecorView`与`contentParent`；
- 在 **installDecor(**) 方法里面，会先判断有没有 `DecorView`，如果没有，先new一个出来，然后判断有没有 `contentParent(承载我们自己布局的ViewGroup)`，没有的话，就去根据当前主题，选择一个布局，并将其当做我们的根布局添加到 `DecorView` 中,再将其中的子view,即R.id.content这个view赋值给我们的 `contentParent`；
- 最后 PhoneWindows-**setContentView()** 方法接下来就可以将我们自己的布局 **inflate** 进这个根布局的 `contentParent` 里了；





