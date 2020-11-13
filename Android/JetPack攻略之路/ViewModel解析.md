# 大朽不工-ViewModel解析

## 概述

### 什么是ViewModel?

ViewModel旨在以注重生命周期的方式存储和管理界面相关的数据，ViewModel 类让数据可在发生屏幕旋转等配置更改后继续留存。

### ViewModel的本质

说简单点，ViewModel 的本质是维护 Ui 的状态，追根到底就是维护对应的数据，无论是MVP 还是 MVVM，UI的展示就是对数据的一个渲染。而ViewModel 就是一个**安全的数据持有类**。

它的做法如下：

1. 定义了 ViewModel 的基类，并建议通过持有LivewData 维护保存数据的状态;
2. ViewModel 不会随着Activity 的 屏幕旋转而销毁，减少了维护状态的代码成本（数据的存储和读取，序列化和反序列化）;
3. 在对应的作用域内，保证只生产出对应的唯一示例，多个Fragment 维护相同的数据状态，极大减少了UI组件之间数据传递的代码成本;

### ViewModel生命周期

![image-20201107194500105](https://tva1.sinaimg.cn/large/0081Kckwly1gkgujifoc5j30by0dzdi8.jpg)



## 使用方式

ExampleActivity

```kotlin
class ExampleActivity : AppCompatActivity() {

    private val viewModel by lazy {
        ViewModelProvider(this).get(ExampleViewModel::class.java)
    }
    
    //确保导入activity-ktx
    // implementation 'androidx.activity:activity-ktx:1.1.0'
    private val viewModelBy by viewModels<ExampleViewModel>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_test)
        viewModel.sumLiveData.observe(this) {

        }
    }
}
```

ExampleViewModel

```kotlin
class ExampleViewModel : ViewModel() {
    private val _sumMutableLiveData = MutableLiveData<String>()
    val sumLiveData: LiveData<String> = _sumMutableLiveData
}
```



## 思路分析

![img](https://user-gold-cdn.xitu.io/2020/3/1/17096a7af51d4c32?imageslim)

带着问题去学习：

1. ViewModel如何被创建出来？
2. ViewModel如何做到数据不被销毁？
3. 如何做到在Fragment和Activity同时使用一个ViewModel?
4. ViewModel在内存不足时如何做到数据依然不会重建？

### ViewModel如何被创建？

```kotlin
 ViewModelProvider(this).get(ExampleViewModel::class.java)
```

#### ViewModelProvider

```java
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
  	//判断传过来的 owner 到底是Fragment还是Activity
    this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
            ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
            : NewInstanceFactory.getInstance());
}

//走到了这里
public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
}
```

#### ViewModelProvider.get(class)

```java
//获取指定的ViewModel对象
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}
```

#### ViewModelProvider.get(key,class)

```java
@NonNull
@MainThread
public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
    ViewModel viewModel = mViewModelStore.get(key);
		//先从map中去取
    if (modelClass.isInstance(viewModel)) {
        if (mFactory instanceof OnRequeryFactory) {
            ((OnRequeryFactory) mFactory).onRequery(viewModel);
        }
        return (T) viewModel;
    } else {
        //noinspection StatementWithEmptyBody
        if (viewModel != null) {
            // TODO: log a warning.
        }
    }
  	//
    if (mFactory instanceof KeyedFactory) {
        viewModel = ((KeyedFactory) (mFactory)).create(key, modelClass);
    } else {
        viewModel = (mFactory).create(modelClass);
    }
    mViewModelStore.put(key, viewModel);
    return (T) viewModel;
}
```



#### Owner.getViewModelStore

![image-20201108120654791](https://tva1.sinaimg.cn/large/0081Kckwly1gkhmx5b8whj30hf0bojt5.jpg)

