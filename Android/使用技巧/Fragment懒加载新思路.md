# Android Fragment懒加载新思路

> 在Android  x以前，我们实现懒加载通常是通过 **setUserVisibleHint** 方法来控制Fragment是否可见。在Android x之后，Google 提供了新的方案给我们。今天我们就来学习一下。



如果直接使用以前的方案，会提示如下，方法已过时：

![image-20191203220602868](https://tva1.sinaimg.cn/large/006tNbRwly1g9jvxhih7gj30yq04st9g.jpg)



点进去查看注释：

![image-20191203221003196](https://tva1.sinaimg.cn/large/006tNbRwly1g9jw1m0xp3j31fq0jowjp.jpg)

大概就是这个方法可以告诉当前Fragment的是否对用户可见，但是可以在生命周期外调用，所以生命周期的方法回调可能不是很准确。所以在Android x里面，Google推荐我们使用 **setMaxLifecycle**.



我们点进去看一下这个方法：

```java
/**
 * Set a ceiling for the state of an active fragment in this FragmentManager. If fragment is
 * already above the received state, it will be forced down to the correct state.
 *
 * <p>The fragment provided must currently be added to the FragmentManager to have it's
 * Lifecycle state capped, or previously added as part of this transaction. The
 * {@link Lifecycle.State} passed in must at least be {@link Lifecycle.State#CREATED}, otherwise
 * an {@link IllegalArgumentException} will be thrown.</p>
 *
 * @param fragment the fragment to have it's state capped.
 * @param state the ceiling state for the fragment.
 * @return the same FragmentTransaction instance
 */
@NonNull
public FragmentTransaction setMaxLifecycle(@NonNull Fragment fragment,
        @NonNull Lifecycle.State state) {
    addOp(new Op(OP_SET_MAX_LIFECYCLE, fragment, state));
    return this;
}
```

通过这个方法，我们可以设置Fragment的生命周期上限，也就是你可以设置这个Fragment最大生命周期限制，如果生命周期执行超过了设置的片段，则会强制降至正确的生命周期。而且传入的生命周期最低限制必须是 Create,也就是执行到了onCreateView。



一共提供了如下几种状态：

![image-20191203221942620](https://tva1.sinaimg.cn/large/006tNbRwly1g9jwbo2bnej30ui0u0alw.jpg)





**下面我们用一个Demo演示一下：**

**xml**

```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.cloud.fragment.TestActivity">

</FrameLayout>
```

**Activity:**

```java
  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        StatusBarUtil.setTranSparentBar(this);
        setContentView(R.layout.activity_test);
        Fragment1 fragment1 = Fragment1.newInstance();
        getSupportFragmentManager()
                .beginTransaction()
                .replace(R.id.fragment, fragment1)
                .setMaxLifecycle(fragment1, Lifecycle.State.CREATED)
                .commit();
        
    }
```

**Fragment1**

```JAVA
public class Fragment1 extends BaseFragment {
 		....

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("demo", "f1-onStart");
    }
}
```

上面的代码都很简单，不必要的地方就省略了，主要是 **setMaxLifecycle** 的使用地方注意即可：



运行我们的模拟器：

![image-20191203223510187](https://tva1.sinaimg.cn/large/006tNbRwly1g9jwrr4ud2j30dg0s60v1.jpg)



因为没有执行到onResume，所以我们的界面无法看到.

log如下：

![image-20191203224301513](https://tva1.sinaimg.cn/large/006tNbRwly1g9jwzx0iadj31180863zj.jpg)





此时更改 为 **Resume**.

```java
   getSupportFragmentManager()
                .beginTransaction()
                .add(R.id.fragment, fragment1)
                .setMaxLifecycle(fragment1, Lifecycle.State.RESUMED)
                .commit();
```

![image-20191203223934927](https://tva1.sinaimg.cn/large/006tNbRwly1g9jwwcakqpj30dg0s6di9.jpg)

日志如下：

![image-20191203224341578](https://tva1.sinaimg.cn/large/006tNbRwly1g9jx0lxkapj311w0ccwge.jpg)

效果如上，使用的方法也没有什么说的了。

其余的几个也不用太说了，都差不多。





#### 现在我们来看一下 **懒加载** 的新方案：

我们先查看一下 **FragmentPagerAdapter** 类：

![image-20191203234946009](https://tva1.sinaimg.cn/large/006tNbRwly1g9jyxd0x3lj30zy05iaaw.jpg)

新增了一个参数，这个参数用来控制选用哪种模式，现在提供了两种模式，如下：



其中，默认使用的是 **BEHAVIOR_SET_USER_VISIBLE_HINT**，至于它们是干什么的，我们具体继续往下看：*当然结合我们开始时的Demo，从注释中也能看出端倪。*

![image-20191203235121500](https://tva1.sinaimg.cn/large/006tNbRwly1g9jyz10rb6j31d80k4wkn.jpg)



因为Android x废除了 **setUserVisibleHint**() 方法，所以相应的适配器也进行了改动：

![image-20191203234030654](https://tva1.sinaimg.cn/large/006tNbRwly1g9jynqkkkmj31fm0s4q9w.jpg)

这里可以看到，上面新增的那两个参数用到了这里。

我们来看里面的逻辑，默认会使用  **Lifecycle.State.RESUMED**，也就是说默认会执行生命周期方法到onResume,并且代替了 **setUserVisibleHint(true)** 方法,而它的作用是用来通知 当前Fragment 是否对用户可见。然后用 **BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT** 代替了 **setUserVisibleHint(false)**。

从而默认的**FragmentPagerAdapter**构造方法会使Fragment 执行到 **onResume**,而指定了 构造参数**BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT** 的，则只会执行到 onStart。



有了这个特性，我们来试试新的懒加载方案：

demo也很简单，只需要new FragmentAdapter时，传入**BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT**即可。

```java
new FragmentAdapter(getSupportFragmentManager(),FragmentPagerAdapter.BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT,list)
```

具体的代码就不用演示了。



下面我们通过log看一下日志变化。

***测试demo：ViewPager-Fragment1-Fragment2***

打开Activity时：

![image-20191204001154967](https://tva1.sinaimg.cn/large/006tNbRwly1g9jzklz2jbj31b20agq5w.jpg)

当切换到第二个Fragment时：

![image-20191204001225348](https://tva1.sinaimg.cn/large/006tNbRwly1g9jzl1i5a4j31di0a8gp2.jpg)



可以观察到每次都会执行onResume,所以我们可以将我们的数据加载方法放在 onResume中。简单的Demo如下：

```java
public abstract class BaseFragment extends Fragment {

    private View view;

    public abstract int setView();

    public abstract void initView();

    public abstract void initData();

    private volatile boolean onVisible = false;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        view = inflater.inflate(setView(), null);
        onVisible = true;
        initView();
        return view;
    }


    @Override
    public void onResume() {
        super.onResume();
      	//
        if (onVisible) {
            initData();
            onVisible = false;
        }
        updateData();
    }
  
   @Override
    public void onPause() {
        super.onPause();
      	//	
    }

    private void updateData() {

    }
    

    public View getView() {
        return view;
    }
}
```

所以我们具体的操作就可以放到 onResume和onPause中，分别对应可见或不可见，很简单。



