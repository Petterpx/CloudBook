# LiveData源码解析

LiveData是google官方架构组件JetPack系列的一个响应式开发框架，其是一种可观察的数据存储器，与常规的可观察类不同，LiveData具有生命周期感知能力，这样感知能力可确保LiveData仅更新处于活跃生命周期状态的应用组件观察者。

LiveData的特点：

- 采用观察者模式自动提示UI更新
- 无需手动处理生命周期,配合ViewModel不会因Activity销毁重建后而丢失数据；
- 不会出现内存泄漏；
- 不需要手动取消订阅，Activity只会在onStart-onPause之间收到消息，需要注意observeForever会一直收到消息，在某些情况下，可能需要手动取消订阅。

## 简要总结流程

- observe

  **添加观察者**

  首先判断当前是否处于主线程，然后判断当前活动是否已经销毁，如果没有继续，接着注册一个生命周期绑定者 LifecycleBoundObserver，然后存到mObservers 这个链表中，如果当前生命周期重复添加，则返回的观察者不为null，如果不为null则抛出异常，否则添加到当前的观察者中。

- observeForever

  **同样也是添加观察者，不同点生命周期非活跃状态也可以收到消息**

- setValue

  **更新LiveData-value并通知观察者，需要在主线程调用**

  当我们调用 setValue 发送数据时，首先会进入更新value中的数据，并将其版本+1，接着进入消息分发dispatchingValue,内部是一个while循环，会去循环调用 considerNotify 也就是我们的通知方法，considerNotify 内部会去先判断当前观察者是否活跃，当前要发送的消息是不是最新的，如果都满足就调用相应的通知方法 onChanged。于是我们的观察者就收到了消息。

- postValue

  **更新LiveData-value并通知观察者，可以在子线程调用**

  postValue内部最终是调用了setValue，不同之处在于，它支持从后台线程调用，因为其内部帮我们切回了主线程再去发送。



## 详细源码分析

### observe

```kotlin
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
  //判断是否主线程  
  assertMainThread("observe");
   //判断当前UI状态如果已经销毁，则不绑定了
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // ignore
        return;
    }
  	//注册一个生命周期绑定观察者
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
  	//存到一个链表中，SafeIterleMap，内部是链表
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
  	//如果已关联过，就抛出异常
    if (existing != null && !existing.isAttachedTo(owner)) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }
  	//和wrapper 与 Activity/Fragment的生命周期建立关系
    owner.getLifecycle().addObserver(wrapper);
}
```

**总结：**首先判断当前是否处于主线程，然后判断当前活动是否已经销毁，如果没有继续，接着注册一个生命周期绑定者 LifecycleBoundObserver，然后存到mObservers 这个链表中，如果当前生命周期重复添加，则返回的观察者不为null，如果不为null则抛出异常，否则添加到当前的观察者中。



### 生命周期变化时

我们接着上面来看一下，当一个生命周期发生变化的时候，LifecycleBoundObserver都做了哪些操作。当我们生命周期变化时，会调用 相应的 **onStateChanged**

如下源码所示：

```java
class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver {
		...
    @Override
    // 当生命周期发生变化时，会调用这个函数 
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
       // 当UI的生命周期为DESTROYED，取消对数据变化的监听，移除回调函数
        if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
            removeObserver(mObserver);
            return;
        }
        //改变数据，传递的参数是shouldBeActive()，它会计算看当前的状态是否是STARTED，也就是 onStart-onPause 期间生命周期
        activeStateChanged(shouldBeActive());
    }
	...
}
```
然后我们继续看 `activeStateChanged` 方法如何对数据进行处理，它是 ObserverWrapper类的一个方法。

```java
void activeStateChanged(boolean newActive) {
        // 当前的生命周期和上一次的生命周期状态，是否发生变化，没有发生变化，就直接返回。
        // onStart-onPause 为 true  在这之外的生命周期为false
        if (newActive == mActive) {
            return;
        }
        // immediately set active state, so we'd never dispatch anything to inactive
        // owner
        mActive = newActive;
        boolean wasInactive = LiveData.this.mActiveCount == 0;
        LiveData.this.mActiveCount += mActive ? 1 : -1;
        if (wasInactive && mActive) {
        // 这是一个空函数，可在代码中根据需要进行重写
            onActive();
        }
        if (LiveData.this.mActiveCount == 0 && !mActive) {
         // 这是一个空函数，可在代码中根据需要进行重写
            onInactive();
        }

		//结合上面的状态判断，我们知道了，生命周期状态从Inactive 到 Active， 就会调用回调函数
        if (mActive) {
            dispatchingValue(this);
        }
    }
```
**为什么从非活跃状态 `Inactive` 到活跃状态 `Active`,要去调用 liveData 设置的回调函数呢？**

原因是，在Inactive(非onstart-onPause周期内)，数据发生了变化，然后在回到Active(onStart-onPause周期内)，如果不去调用回调函数，会出现UI的界面，还在显示上一次的数据，所以调用回调函数。

再回到代码流程，我们看一下调用 dispatchingValue(this),在发生变化后，同样会去通知当前以及绑定的观察者。

### setValue

```java
protected void setValue(T value) {
  	//判断是否是主线程
    assertMainThread("setValue");
  	//当前版本+1，表示数据发生了变化
    mVersion++;
  	//更新当前变化的数据
    mData = value;
  	//开始分发消息
    dispatchingValue(null);
}
```

进入 **dispatchingValue** 方法

该方法是通过一个 while 循环来通知回调函数：

1. 如果调用 `dispatchingValue` 方法，传入参数，则调用指定的liveData回调函数；
2. 如果调用 `dispatchValue` 方法，传入null,则在for循环中，依次通知所有livedata的回调函数；

```java
  @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
      	//是否正在发送消息
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
            return;
        }
      	//
        mDispatchingValue = true;
        do {
          	//开始循环前，设置为false,for循环完，也会退出while循环
            mDispatchInvalidated = false;
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
              	//循环调用观察者
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                  	//这里 mDispatchInvalidated 为true,表示在while循环未结束的时候，如果有其他数据发生变化，并调用了该函数，则在上面的if判断中设置 mDispatchInvalidated=true,
                  	//结束本次循环，没有退出while循环，开始下一次for循环
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
```

接着来看 **considerNotify** 方法。

```java
@SuppressWarnings("unchecked")
private void considerNotify(ObserverWrapper observer) {
  	//判断lifecycle是否处于活跃状态
    if (!observer.mActive) {
        return;
    }
    // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
    //
    // we still first check observer.active to keep it as the entrance for events. So even if
    // the observer moved to an active state, if we've not received that event, we better not
    // notify for a more predictable notification order.
  	//再次检查最新的活动状态，可能消息已发送
    if (!observer.shouldBeActive()) {
        observer.activeStateChanged(false);
        return;
    }
  
  	//判断消息版本，如果最新消息大于要发生的消息版本，就撤销这次发送
    if (observer.mLastVersion >= mVersion) {
        return;
    }
  	//更新最新消息
    observer.mLastVersion = mVersion;
  	//通知观察者
    observer.mObserver.onChanged((T) mData);
}
```

**总结:**当我们调用 setValue 发送数据时，首先会进入更新value中的数据，并将其版本+1，接着进入消息分发dispatchingValue,内部是一个while循环，会去循环调用 considerNotify 也就是我们的通知方法，considerNotify 内部会去先判断当前观察者是否活跃，当前要发送的消息是不是最新的，如果都满足就调用相应的通知方法 onChanged。于是我们的观察者就收到了消息。



### postValue

```java
protected void postValue(T value) {
    boolean postTask;
    synchronized (mDataLock) {
        postTask = mPendingData == NOT_SET;
        mPendingData = value;
    }
  	//上一个post后还没有执行的runnable，就不需要再 post了
  	// 上面的mPendingData数据已经是新数据了
    if (!postTask) {
        return;
    }
  	//postValue可以从后台线程调用，因为其最终会在主线程执行任务
    ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
}
```

```java
private final Runnable mPostValueRunnable = new Runnable() {
    @SuppressWarnings("unchecked")
    @Override
    public void run() {
        Object newValue;
        synchronized (mDataLock) {
            newValue = mPendingData;
            mPendingData = NOT_SET;
        }
      	//最后依然调用了setValue
        setValue((T) newValue);
    }
};
```

**总结**

postValue内部最终是调用了setValue，不同之处在于，它支持从后台线程调用，因为其内部手动帮我们将数据回调到主线程再去发送。