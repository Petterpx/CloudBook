# JetPack组件攻略-lifecycle

##### lifecycle

简介：它是用来管理生命周期的组件。

使用场景：我们一般在写代码时，都不可避免的需要重写某些生命周期，造成代码的冗余与复杂，而借助lifecycle就可以将生命周期方法扩展，从而避免直接重写相应方法。这对我们写框架来说，有很大简洁优化作用。它是基于观察者模式的实现。

使用方法。



先绑定生命周期。需要相应的类实现DefaultLifecycleObserver接口并实现相应的扩展方法。

```java
LifecycleOwner//  getLifecycle().addObserver(presenter);
```

销毁时：

```java
LifecycleOwner//  getLifecycle().removeObserver(this);
```

需要注意的是Activity与Fragment现在已经可以直接使用getLifecycle()方法。