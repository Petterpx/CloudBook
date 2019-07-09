## RxJava--初识

## 概述

### 什么是响应式编程？

是一种基于异步数据流概念的编程模式。

### 为什么要用 Rxjava

在异步操作中，我们会想到Android 的 AsyncTask和 Handler ，但是随着请求数量越来越多，代码逻辑将会变得越来越复杂，而 RxJava 却仍旧能保持清晰的逻辑。RxJava 的原理就是创建一个 Observable 对象来干活，然后使用各种操作符建立起来的链式操作，就如同流水线一样，把你想要处理的数据一步一步的加工成你想要的成品，然后发射给 Subscriber 处理。

### RxJava 与 观察者模式

RxJava 的异步操作是通过扩展的观察者模式来实现的，RxJava有4个角度 Observale,Observer,Subscriber 和 Suject。其中 Observable 和 Observer 通过 subscrible 方法实现订阅关系，Observable 可以在需要的时候通知 Observer.

### RxJava 基本实现

#### 1.创建 Observer(观察者)

```java
 Subscriber subscriber=new Subscriber<String>() {
        @Override
        public void onSubscribe(Subscription s) {
            Log.e("demo","onSubscribe");
        }

        @Override
        public void onNext(String s) {
            Log.e("demo","onNext");
        }


        @Override
        public void onError(Throwable t) {
            Log.e("demo","onError");
        }

        @Override
        public void onComplete() {
            Log.e("demo","onComplete");
        }
        
    };
}
```

必须实现以上4种方法。其含义如下：

- onComplete:事件队列完结。RxJava不仅把每个事件都单独处理，其还会把它们看做一个队列。当不在会有新的 onNext 发出时，需要触发 onComplete() 方法作为完成标记。
- onError: 事件队列异常。在事件处理过程中出现异常时，onError()会被触发，同时队列自动终止，不允许再有事件发生。
- onNext: 普通的事件。将要处理的时间添加到事件队列中。
- onSubscribe: 订阅时调用。

#### 2.创建被观察者：

```java
Observable observable=Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onComplete();
    }
});
```

通过调用 Subcriber 的方法，不断将事件添加到任务队列中，也可以用 just 方法来实现：

```java
Observable observable=Observable.just("1","2","3");
```

上述代码会依次调用 onNext("1"),... onComplete().还可以用 from方法来实现，如下所示：

```java
String[] words={"1","2","3"};
Observable observable=Observable.fromArray(words);
```

其调用规则与上面哪些是一致的。



#### 3.订阅

```java
observable.subscribe(observer);
```

只需要上面一行代码即可。

上面这些步骤其实也可以使用链式调用：

```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
})
.subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {

    }

    @Override
    public void onError(Throwable e) {

    }

     @Override
     public void onComplete() {

     }
        });
```



#### 4.执行结果

```java
2019-07-06 21:46:53.111 16317-16317/com.petterp.test2 E/demo: onSubscribe
2019-07-06 21:46:53.111 16317-16317/com.petterp.test2 E/demo: 1
2019-07-06 21:46:53.111 16317-16317/com.petterp.test2 E/demo: 2
2019-07-06 21:46:53.111 16317-16317/com.petterp.test2 E/demo: 3
2019-07-06 21:46:53.111 16317-16317/com.petterp.test2 E/demo: onComplete
```

可以看出，先调用 onSubscribe,然后再调用三个 onNext方法，最后调用 onCompleted.

**被观察者发送事件有以下几种，总结如下：**

| 事件种类     | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| onNext()     | 发送该事件时，观察者会回调 onNext() 方法                     |
| onError()    | 发送该事件时，观察者会回调 onError() 方法，当发送该事件之后，其他事件将不会继续发送 |
| onComplete() | 发送该事件时，观察者会回调 onComplete() 方法，当发送该事件之后，其他事件将不会继续发送 |

其实我们把RxJava 想象成一个加工商，你自己有很原始材料(要发送的原始数据),你想制作出某个工具，这时候你就得知道具体究竟要什么样的工具。比如你如果想要个菜刀，那就需要刀把，刀刃(使用各种操作符变换你想发送给观察者的数据)，制作完成后，你就可以使用你想要的工具了(把处理好的数据发送给观察者)。

总结如下图：

![img](E:/Android_NoteBook/Android_NoteBook/assets/1639a8ee56b13c41)

