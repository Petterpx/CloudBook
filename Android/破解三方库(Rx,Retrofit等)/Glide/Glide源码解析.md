# 深入分析Glide源码 - 解构篇

本文是深入分析Glide源码第一篇，解构，也是最基础的一篇，本文将带你明白Glide最基本的一些实现细节。

> 深入分析Glide源码三部曲 - 解构篇
>
> 深入分析Glide源码三部曲 - 入境
>
> 深入分析Glide源码三部曲 - 贯穿思路
>
> 深入分析Glide后，轻松完虐面试官的问题
>
> 深入分析Glide后，如何才能自己写一个图片加载框架 - 分析思路
>
> 深入分析Glide后，我动手写了一个图片加载库 - 实战篇

> 本文基于Glide最新4.12

Glide是我们最常用的一个图片解析框架，相信每一个Android开发同学都不会陌生，其使用也非常简单，但是Glide又与Okhttp这种框架不同，因为其整体复杂性相比后者不知大了几个体量级，理解起来也并不是那么容易。

当我们使用Glide时，我们经常是这样一个操作，最简单情况下：

```kotlin
Glide.with(this).load(url).into(image)
```

那么上述这段代码内部又做了哪些事呢？

在开始源码解析前，我们先明白这几个类的主要作用是什么：

#### RequestManager

网络请求管理对象。

RequestBuilder



话不多说，让我们从源码中开始着手。

##  Glide.with

即绑定生命周期阶段。

![image-20210928001858475](https://tva1.sinaimg.cn/large/008i3skNly1guvn8fintwj60r407mt9g02.jpg)

一个简单的Glide.witch,其包含了6种重载，具体如上所述。上面这6种，我们按照具体，可分为3类，Fragment,View,Activity。

##### with(Fragment)

with(Fragment)对应了两种，一种是旧版App包下的Fragment,另一种也就是现在主流的AndroidX包下：

```kotlin
fun with(android.app.Fragment):RequestManager
fun with(Fragment):RequestManager
```

##### with(Activity)

with(Activity)同样也对应了两种，前者是Activity,后者是FragmentActivity

```kotlin
fun with(Activity):RequestManager
fun with(FragmentActivity):RequestManager
```

##### with(View)

适用于生命周期位于View上

##### with(Context)

适用与Application级别的。



### Fragment处理方式

#### RequestManagerRetriever.get(Fragment)

```java
public RequestManager get(@NonNull Fragment fragment) {
  //判断是否是后台线程
  if (Util.isOnBackgroundThread()) {
    return get(fragment.getContext().getApplicationContext());
  } else {
    // 注册此Fragment,某些场景下，Fragment不由Activity托管，相当于给一个入口，便于自己管理
    if (fragment.getActivity() != null) {
      frameWaiter.registerSelf(fragment.getActivity());
    }
    // 获取FragmentManager
    FragmentManager fm = fragment.getChildFragmentManager();
    // 获取生成的RequestManager,即管理器
    return supportFragmentGet(fragment.getContext(), fm, fragment, fragment.isVisible());
  }
}
```



#### RequestManagerRetriever.supportFragmentGet()

```java
@NonNull
private RequestManager supportFragmentGet(
    @NonNull Context context,
    @NonNull FragmentManager fm,
    @Nullable Fragment parentHint,
    boolean isParentVisible) {
  // 获取专门用于管理此次请求的Fragment -> 1.
  SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint);
  // 获取此Fragment绑定的manager 
  RequestManager requestManager = current.getRequestManager();
  //如果为null,开始初始化
  if (requestManager == null) {
    // 获取Glide对象，get方法内部会使用双重检查锁。 -> 2
    Glide glide = Glide.get(context);
    // 初始化一个manager，-> 3
    requestManager =
        factory.build(
            glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
    
    //这里先启动requestManager的start方法，而不是lifecycle,目前是避免内存泄漏
    if (isParentVisible) {
      requestManager.onStart();
    }
    current.setRequestManager(requestManager);
  }
  return requestManager;
}
```

##### getSupportRequestManagerFragment()

```java
private SupportRequestManagerFragment getSupportRequestManagerFragment(
    @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
  SupportRequestManagerFragment current =
      (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
  if (current == null) {
    current = pendingSupportRequestManagerFragments.get(fm);
    if (current == null) {
      current = new SupportRequestManagerFragment();
      current.setParentFragmentHint(parentHint);
      pendingSupportRequestManagerFragments.put(fm, current);
      fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
      handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
    }
  }
  return current;
}
```





我将Glide的分析分为3篇，第一篇主要解释Glide的几个比较大的api,以及with里面做了什么，以及glide如何做到什么时候再去加载图片，什么时候停止加载。







## 源码解析



