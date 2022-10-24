# ViewPager中Fragment状态保存的哪些事

**Hi** ，很高兴见到你！ 

## 引言

在使用 `ViewPager` 时 , 如果我们的适配器使用的是 `FragmentStatePagerAdapter` ，那么当我们重新滑到之前已销毁的页面时，一般情况下页面的状态依然将保持不变(比如 RecyclerView 的 **滚动位置**等,EditText 的 **输入内容** 等), 或者说 View 历史状态被还原了。

本文的主旨就是解释其 **保存与还原内部的原理以及过程**。

## 基础概念

ViewPager 官方的适配器有两种，即 `FragmentPagerAdapter` 以及 `FragmentStatePagerAdapter` 。前者适用于少量Item时，后者适用于多个item。
> 
> 主要原因是 `FragmentStatePagerAdapter` 每次会重建以及 **销毁** Fragment, 而 `FragmentPageAdapter` 并不会销毁实例，只是对视图做了 **attach** 和 **detach** 。

## 举个 🌰

如下段代码所示，我们有这样一个适配器 [MainAdapter]：

```kotlin
class MainAdapter(fragmentManager: FragmentManager, private val datas: List<String>) :
    FragmentStatePagerAdapter(fragmentManager) {
    override fun getCount(): Int {
        return datas.size
    }

    override fun getItem(position: Int): Fragment {
        return T1Fragment.newInstance(datas[position])
    }
}
```

其余代码比较简易,我们用以下层级即可代表：

```kotlin
MainActivity 
 ViewPager(adapter = MainAdapter , offscreenPageLimit = 1)
 		Fragment(key) - (by activityViewModel)
 				RecyclerView - (data = activityViewModel.data[key])
```

> 如上所示，我们有一个 Activity,其内部有一个 ViewPager,ViewPager 的适配器就是我们上面写的 MainAdapter，默认缓存 **n(1)+2** 。
>
> Fragment 内部是一个 RecyclerView，其数据源来自 **activity级** 的ViewModel(即我们对数据根据key做了缓存,避免每次的重新初始化)

我们做一个滚动测试,然后再看看 Fragment 重新创建后 View状态(**RecyclerView滚动位置**) 的变化，如下所示：

<div align=center><img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a15c97b3c5b4f78aefd2193c91fd8ec~tplv-k3u1fbpfcp-zoom-1.image" alt="iShot2022-04-13_23.25.57" width="300"/></div>

因为默认缓存为 **n(1)+2** ,即当我们滑动到 item=3 时，**1** 页面此时已被销毁。但当我们重新切换到 **1** 时，可以发现，Fragment1 中 `RecyclerView` 的 **滚动位置** 没有变化，所以可以证明 Fragment 的状态的确是被还原了。

**那这是怎么做的呢？** 带着这个问题，我们开始比较简单的源码解析环节。

## Adapter解析

直接去看 `FragmentStateAdapter`

```java
  ...
  // 保存Fragment的状态list
  private ArrayList<Fragment.SavedState> mSavedState = new ArrayList<>();
	...
```

其内部有一个名为 `mSavedState` 的List,用于保存我们的 Fragment状态 ,那这个 `mSavedState` 又会在哪里被调用呢？既然要还原以及保存，那就免不了两个地方，**[初始化]** 与 **[销毁]** ,所以我们继续往下去看 `instantiateItem()` 与 `destroyItem()`。

### destroyItem()

> 此方法用于销毁我们的指定Fragment,其内部把当前Fragment的状态根据下标保存到了 mSavedState 中。

```java
public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    Fragment fragment = (Fragment) object;
		...
    // 避免数组长度不足导致的越界异常
    while (mSavedState.size() <= position) {
        mSavedState.add(null);
    }
  	// 调用 mFragmentManager 去保存Fragment 的状态,并将其保存在了内部的 mSavedState 中
    mSavedState.set(position, fragment.isAdded()
            ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
  	// 销毁fragment
    mFragments.set(position, null);
		...
}
```

### instantiateItem()

> 此方法主要用于初始化 **指定position** 对应的 Fragment 。
>
> 在初始化 Fragment 时，其会通过 下标position 从  `mSavedState` 找到缓存的 Fragment 状态，然后将设置给其，便于后续的使用。

```java
public Object instantiateItem(@NonNull ViewGroup container, int position) {
  	// 如果fragment已存在直接返回
    if (mFragments.size() > position) {
        Fragment f = mFragments.get(position);
        if (f != null) {
            return f;
        }
    }
		// 初始化Fragment,在adapter中，我们需要重写此方法，实现我们的Fragment初始化
    Fragment fragment = getItem(position);
    if (DEBUG) Log.v(TAG, "Adding item #" + position + ": f=" + fragment);
  	// 数组健壮性保护
    if (mSavedState.size() > position) {
      	// 获取指定位置保存的状态
        Fragment.SavedState fss = mSavedState.get(position);
        if (fss != null) {
          	// 将状态重新设置给fragment
            fragment.setInitialSavedState(fss);
        }
    }
   	...
    return fragment;
}
```

### 小结

所以我们可以简单理解为 `FragmentStatePagerAdapter` 之所以可以做到状态还原,是因为其在销毁 Fragment 时，默认缓存了当前 Fragment 的状态信息，并且以下标的方式进行了保存，当我们在滑动 ViewPager 时，其会加载并初始化指定 position 所对应 Fragment ,并将缓存的 Fragment 的状态信息 set 进去。

## Fragment部分

通过上面的方式，我们可以简单的知道 ViewPager 是如何帮我们进行状态还原与保存，那 Fragment 到底是在什么时候去使用这个状态呢？所以带着这个问题，我们接着去看看 Fragment 的源码。

无论是 View 还是 Fragment ，其都具有 这个方法 `onSaveInstanceState` ,既然有保存的方法，那肯定也有还原的方法。

在Fragment中我们去看这个方法：`onViewStateRestored()`

> 官方解释，此方法被调用时意味着 Fragment所有状态 都已经还原。

所以我们直接去看看到底是在哪里调用了此方法，也就知道 Fragment 是怎么还原状态的。具体的调用栈如下：

> 1. FragmentManager - moveToState() 👇🏻
> 2. FragmentManager - activityCreated() 👇🏻
> 3. ​     Fragment - performActivityCreated() 👇🏻
> 4. ​     Fragment - restoreViewState() 👇🏻
> 5. ​     Fragment - restoreViewState(Bundle) 👇🏻

### FragmentManager

```java
// 1.
// fragment状态变化
void moveToState(){
   switch (f.mState) {
       // 当view已经创建好时
   		 case Fragment.VIEW_CREATED:  fragmentStateManager.activityCreated();
   }
}
 
// 2. 通知活动已创建
void activityCreated() {
  	// 执行fragment的 ActivityCreated 方法,相当于fragment与act已绑定
        mFragment.performActivityCreated(mFragment.mSavedFragmentState);
  	// 调度Fragment的生命周期
        mDispatcher.dispatchOnFragmentActivityCreated(
                mFragment, mFragment.mSavedFragmentState, false);
 }
```

### Fragment

```java
// 3. 执行与act绑定时的逻辑
void performActivityCreated(Bundle savedInstanceState) {
        mChildFragmentManager.noteStateNotSaved();
        mState = AWAITING_EXIT_EFFECTS;
        mCalled = false;
  			// 触发
        onActivityCreated(savedInstanceState);
       	...
        restoreViewState();
        mChildFragmentManager.dispatchActivityCreated();
}

// 4. 恢复视图状态
private void restoreViewState() {
       	if (mView != null) {
            restoreViewState(mSavedFragmentState);
        }
        mSavedFragmentState = null;
}

// 恢复具体的视图状态
final void restoreViewState(Bundle savedInstanceState) {
    // 视图状态不为null,则恢复之前的视图层级
    if (mSavedViewState != null) {
        mView.restoreHierarchyState(mSavedViewState);
        mSavedViewState = null;
    }
    if (mView != null) {
        mViewLifecycleOwner.performRestore(mSavedViewRegistryState);
        mSavedViewRegistryState = null;
    }
    mCalled = false;
    // 通知view的状态已被还原
    onViewStateRestored(savedInstanceState);
		..
}
```

## 总结

当我们使用 ViewPager 时，如果使用 `FragmentStatePagerAdapter` 作为适配器，Fragment 的状态会被主动还原，主要原因是：

- Fragment 销毁时，会调用 `destoryItem` 方法，adapter内部会主动保存了当前的 Fragment 状态，并以当前下标作为 `key` 存到了一个list集合中，然后在调用 `getItem()` 初始化Fragment时，其会将之前保存的状态重新 **set** 给我们的 Fragment 实例。
- 当 Fragment 生命周期执行到 `activityCreated` 时，从而调用 `restoreViewState()` 触发View状态的恢复(此时onCreateView已执行)，然后将我们的view状态还原上去。

知道了这个概念，我们也就可以自己做一些小扩展，比如我们可以在部分情况下主动将我们的Fragment状态保存起来，以便在后面进行恢复，也即就是使用以下两个方法即可。

```kotlin
// 保存
FragmentManager.saveFragmentInstanceState(fragment)
// 还原
Fragment.setInitialSavedState(SavedState)
```



## 关于我

> 我是Petterp,一个三流开发，如果本文对你有所帮助，欢迎点赞评论，你的支持是我持续创作的最大鼓励！