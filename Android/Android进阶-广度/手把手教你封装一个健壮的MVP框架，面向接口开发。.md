# 手把手教你封装一个健壮的MVP框架，面向接口开发。

在我们的日常开发中，我们都知道 Android 端的开发框架有 MVC,MVP,MVVM,说起这几个框架，大家也肯定都有自己的看法，甚至很多同学也都封装过。

问题来了：现在不都是 MVVM 了吗，你还写MVP干吗，有用吗，网上那么多轮子，找个 star 高的不就行了。

> 使用和自己动手封装完全是两个过程，需要考虑多方面，这其中需要踩很多坑。目前这个框架已经应用在我写的公司项目上。趁最近有时间，对框架中存在的问题和缺陷，经历过这次实战，也进行了修复，同时加入了一些JetPack的组件。希望对大家有所帮助

首先，简单介绍一下什么是MVP模式。

![image-20191105200707600](https://tva1.sinaimg.cn/large/006y8mN6ly1g8nf554ki9j31060kc0wd.jpg)

简单理解就是：

> P层相当于一个中间商，天天喊着 xxx,不赚差价。。。(日常开发中,P难免会涉及一些逻辑操作，但并不影响什么，不能为了设计模式而去一定要怎么做)
>
> M 层就是一个老实巴交的工人，处理各种苦活，累活
>
> V 层相当于一个小姐姐，负责美貌，所以只负责展示UI
>
> 总结一下过程：小姐姐(V层)要出门了，缺一个口红，找到P层去代购，中间商(P层)收到命令，找到工人小马(M层)去跑腿购买，M层买完之后，告诉中间商，买好了，中间商再去通知小姐姐，然后小姐姐拿到自己的口红，高兴的出门去见小哥哥了。





#### 下面开始我们的代码过程：

在上面说过了，面向接口编程。

### View-接口:

> 首先是V层接口，注意，这些都可以改，框架封装并不一定需要遵循死规则

```java
public interface IView {

    /**
     * 刷新UI时
     */
    default void updateView() {

    }

    /**
     * 获取Context
     *
     * @return
     */
    Context context();

    void showLoader();

    void stopLoader();

    /**
     * 销毁
     */
    default void onDetachView() {

    }

    /**
     * 关闭键盘
     */
    void hidekey();

}

```



### View-超类

> 

```java
/**
 * Fragment基类
 * @author by Petterp
 * @date 2019-08-03
 */
public abstract class BaseFragment<P extends IPresenter> extends Fragment implements IView{
    private P presenter = null;
    private Unbinder unbinder = null;
    private View rootView = null;
    /**
     * 设置view
     *
     * @return view
     */
    public abstract Object setLayout();


    /**
     * 创建视图
     *
     * @param savedInstanceState
     * @param rootView
     */
    public abstract void onBindView(@Nullable Bundle savedInstanceState, @NonNull View rootView);

    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        if (setLayout() instanceof Integer) {
            rootView = inflater.inflate((Integer) setLayout(), container, false);
        } else if (setLayout() instanceof View) {
            rootView = (View) setLayout();
        } else {
            throw new ClassCastException("setLayout() must be int or View Error！");
        }

        if (presenter == null) {
            presenter = (P) PresenterFactoryImpl.createFactory(getClass());
        }

        if (presenter != null) {
            presenter.setView(this);
            getLifecycle().addObserver(presenter);
        }

        //绑定ButterKnife
        unbinder = ButterKnife.bind(this, rootView);
        //Fragment回收保留数据
        setRetainInstance(true);
        //添加生命周期
        onBindView(savedInstanceState, rootView);
        return rootView;
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        setTitleToolbar();
    }

    /**
     * 沉浸式状态栏
     */
    private void setTitleToolbar() {
        ImmersionBar.with(this)
                .titleBar(setToolbar())
                .autoDarkModeEnable(true)
                .init();
    }

    @Override
    public Context context() {
        return getContext();
    }

    /**
     * 设置Toolbar
     *
     * @return
     */
    public View setToolbar() {
        return null;
    }

    /**
     * 返回P
     *
     * @return Presenter
     */
    public P getPresenter() {
        return presenter;
    }

    /**
     * 返回子类View
     *
     * @return view
     */
    protected View getRootView() {
        return rootView;
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        if (unbinder != null) {
            unbinder.unbind();
            unbinder = null;
        }
        presenter = null;
        rootView = null;
    }


    @Override
    public void hidekey() {

    }


    @Override
    public void showLoader() {

    }

    @Override
    public void stopLoader() {

    }
}

```





### Model-接口

> **注意**：框架中集成了RxJava+RxAndroid,这里的 RxModelinit 用于处理一些初始化时的数据加载任务。

```java
public interface IModel<P extends IPresenter> {

    /**
     * 某些初始化行为
     */
    default void initData() {

    }
    /**
     * 设置P
     * @param p
     */
    void setPresenter(P p);

    /**
     * 初始化耗时行为
     */
    default void RxModelinit() {

    }

    /**
     * 获取P层对象
     *
     * @return P
     */
    P getPresenter();
}

```



### Model-超类

> 这里没有什么需要注意的，常规操作。

```java
public abstract class BaseModel<P extends IPresenter> implements IModel {
    /**
     * P层接口
     */
    private P p;

    /**
     * 设置P
     * @param iPresenter
     */
    @Override
    public void setPresenter(IPresenter iPresenter) {
        this.p= (P) iPresenter;
    }

    /**
     * 获取P层接口
     * @return
     */
    @Override
    public P getPresenter() {
        return p;
    }
}

```



### Presenter-接口

> **注意**：超类继承 DefaultLifecycleObserver，原因是赋予生命周期
>
> **注意：**框架中已经集成了rx耗时任务处理方案，根据使用习惯可以动态调整

```java
public interface IPresenter<V extends IView, M extends IModel> extends DefaultLifecycleObserver {

    /**
     * 一些初始化的操作
     */
    default void initPresenter() {

    }

    /**
     * 是否需要LiveBus-> 暂时为EventBus
     *
     * @return
     */
    default boolean isLiveBus() {
        return false;
    }

    /**
     * 设置View
     *
     * @param v
     */
    void setView(V v);

    /**
     * 获取V
     *
     * @return V
     */
    V getView();

    /**
     * 获取M
     *
     * @return M
     */
    M getModel();

    /**
     * 启动Rx初始化耗时任务
     */
    void rxStartInitData();

    /**
     * Rx任务结束时
     */
    default void rxEndInitData() {

    }

    /**
     * rx初始化时
     */
    default void rxSpecificData() {

    }

    /**
     * 是否需要Rx
     *
     * @return mode
     */
    default boolean rxMode() {
        return false;
    }
}
```



### Presenter-超类

> **注意**： 这里用到了动态代理。动态代理的目的是为了避免View空指针，从而减少多次的View判空。
>
> **注意**： 框架中继承了状态栏处理工具 immersionbar，使用时只需实现 setToolbar(), 然后传入相应的view即可。
>
> **注意**：这里用到了 JetPack 生命周期组件，目的是赋予P层生命周期，因为面向接口开发，所以我们的P 层接口需要继承于 DefaultLifecycleObserver。

```java
public abstract class BasePresenter<V extends IView, M extends IModel> implements IPresenter<V, M>, InvocationHandler {

    private SoftReference mView;
    private Disposable subscribe = null;
    private M model;
    private V proxyView;


    @SuppressWarnings("unchecked")
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    @Override
    public void setView(V v) {
        this.mView = new SoftReference(v);
        proxyView = (V) Proxy.newProxyInstance(v.getClass().getClassLoader(), v.getClass().getInterfaces(), this);
        model = (M) ModelFactoryImpl.createFactory(getClass());
        if (model != null) {
            model.setPresenter(this);
        }
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return isAttached() ? method.invoke(mView.get(), args) : null;
    }

    private boolean isAttached() {
        return mView.get() != null && proxyView != null;
    }


    @Override
    public V getView() {
        return proxyView;
    }

    @Override
    public M getModel() {
        return model;
    }


    @Override
    public void rxStartInitData() {
        subscribe = Observable
                .create(emitter -> {
                    rxSpecificData();
                    emitter.onComplete();
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .doOnComplete(this::rxEndInitData)
                .subscribe();
    }


    @Override
    public void onStart(@NonNull LifecycleOwner owner) {

    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
        //初始化一些操作，基本操作
        initPresenter();
        if (model != null) {
            getModel().initData();
        }
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {

    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {
        //关闭键盘,建议添加全局Activity,这里调用公共view层的hidekey方法
        proxyView.hidekey();
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {
        proxyView.onDetachView();
        if (mView != null) {
            mView.clear();
            mView = null;
        }

        //如果开启过EventBus，关闭
        if (isLiveBus()) {
            EventBus.clearCaches();
            EventBus.getDefault().unregister(this);
        }

        //取消Rx订阅
        if (subscribe != null && !subscribe.isDisposed()) {
            subscribe.dispose();
        }
        //取消生命周期
        owner.getLifecycle().removeObserver(this);
    }
}
```



### 注解相关

#### @CreateModel -生成M层

```java
@Inherited //可重复
@Retention(RetentionPolicy.RUNTIME) //运行时
public @interface CreateModel {
    Class<? extends BaseModel> value();
}

```

```java
public class ModelFactoryImpl {
    /**
     * 根据注解创建Presenter的工厂实现类
     *
     * @param viewClazz 需要创建Presenter的V层实现类
     * @param <M>       当前要创建的Model类型
     * @return 工厂类
     */
    @SuppressWarnings("unchecked")
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    public static <M extends BaseModel> M createFactory(Class<?> viewClazz) {

        CreateModel annotation = viewClazz.getAnnotation(CreateModel.class);
        Class<M> aClass = null;
        if (annotation != null) {
            aClass = (Class<M>) annotation.value();
        }
        try {
            return aClass != null ? aClass.newInstance() : null;
        } catch (IllegalAccessException | InstantiationException e) {
            throw new RuntimeException("Presenter创建失败!，检查是否声明了@CreatePresenter(xx.class)注解");
        }
    }
}
```



#### @CreatePresenter -生成P层

```JAVA
@Inherited //可重复
@Retention(RetentionPolicy.RUNTIME) //运行时
public @interface CreatePresenter {
    Class<? extends BasePresenter> value();
}
```

```JAVA
public class PresenterFactoryImpl {

    /**
     * 根据注解创建Presenter的工厂实现类
     *
     * @param viewClazz 需要创建Presenter的V层实现类
     * @param <P>       当前要创建的Presenter类型
     * @return 工厂类
     */
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    public static <P extends BasePresenter> P createFactory(Class<?> viewClazz) {
        CreatePresenter annotation = viewClazz.getAnnotation(CreatePresenter.class);
        Class<P> aClass = null;
        if (annotation != null) {
            aClass = (Class<P>) annotation.value();
        }
        try {
            return aClass != null ? aClass.newInstance() : null;
        } catch (IllegalAccessException | InstantiationException e) {
            throw new RuntimeException("Presenter创建失败!，检查是否声明了@CreatePresenter(xx.class)注解");
        }
    }
}
```





### 具体使用过程如下：

```java
//Presenter类
@CreateModel(TestModels.class)
public class TestPresenter extends BasePresenter<TestControl.testView, TestControl.testModel> implements TestControl.testPresenter {

}
```

```java
//Model
public class TestModels extends BaseModel<TestControl.testPresenter> implements TestControl.testModel{

}
```

```java
//View
@CreatePresenter(TestPresenter.class)
public class TestFragment extends BaseFragment<TestControl.testPresenter> implements TestControl.testView {
    @Override
    public Object setLayout() {
        return R.layout.test_fragment;
    }

    @Override
    public void onBindView(@Nullable Bundle savedInstanceState, @NonNull View rootView) {

    }
    }
```

```java
//Control契约接口
public interface TestControl {
    interface TestView extends IView {

    }

    interface TestPresenter extends IPresenter<testView, testModel> {
    }

    interface TestModel extends IModel {
    }
}
```



***能看到这里，证明这篇文章对你有所帮助，不胜荣幸。***

> **下面我简单谈一下我对移动端框架的想法和一些封装过程中的理解及项目实战中的坑。**



#### MVP架构中，网上有些图中M和P不是没有互相关联吗，为什么你要选择互相关联？

> 在以前的封装中，我并没有让M与P互相关联，但是在实际开发中，我逐渐发现很多地方其实需要用到P，如果不关联，将导致很多逻辑操作必须放在P层，从而弱化了M层真正作用。其实 MVP中，很多人认为P层是最重要的，其实不然，P只是一个中间商，它的作用就是协调M和V，从而让M,V解耦。当然可以处理一些逻辑，甚至你都可以将逻辑放在P层,不能说错了,只能说个人的理解问题。

#### 很多人都在用弱引用P对象或者View对象，这个真有用吗，实际意义？

> 首先，有用，因为P持有View，其次也有一定风险，Java的引用分为4种，具体大家可以百度。如果直接使用弱引用等，都有被回收的风险，所以更好的处理方式是使用引用队列(当对象被回收时，会将对象放进队列)，但同时也要注意，当你get()之后，此时你的引用已经成了强引用。

#### 如果我有一些模块需要复用怎么办？

> 我个人推荐使用策略模式进行改造。就是将相同的方法分离出一个接口，你的主Fragment或者Activity持有这个接口对象，并提供set方法，不同的子类实现这个接口。然后使用时，set进相应的子类实例，然后使用接口对象调用共有方法。简单理解其实就是多态。
>
> 我本来现在框架中加入一个公共的策略超类，但是侵入严重，所以放弃了加入。当然设计模式有很多，有时候没有模式才是最好的模式。

#### 我应该选用MVP还是MVVM？

> 没有最好的，只有最合适的。我想大家听到这句话估计都想骂人，这说的是啥嘛？
>
> 作为一个过来人，我有必要告诉你，不要一开始就接触成熟的MVVM或者MVP框架，因为作者考虑的很多，可能对初学者并不会很友好，所以从基础教程走起。
>
> 最后，其实MVP和MVVM差别不是很大，如何使用取决于你的项目，如果只是学习，那么建议都是用一下，实际开发的话。如果拥有同样的学习时间，我更推荐MVVM，毕竟它其实比MVP要更省事。这句话可能存在歧义，有些同学认为，MVVM很难啊，听起来要学ViewModel,DataBing,LiveData,成本很高啊。但其实并不是这样，越是看起来复杂的东西，往往学会之后很简单。除去DataBing,学会ViewModel+LiveData是非常快的一件事，特别是当你自己封装过MVP之后，就会有一种原来都差不多的感觉。

如果在使用中有什么疑问的地方，欢迎留言,非常希望和大家一起进步，成长。

## 最后，附上链接。[CloudMVP](https://github.com/Petterpx/CloudMVP)