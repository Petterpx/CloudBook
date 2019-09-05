# Android内存泄漏场景及解决方案

**先说一下什么是Java的内存泄漏？**

> 在Java中，内存泄漏就是存在一些被分配的对象，这些对象有下面两个特点，首先，这些对象是可达的，即在有向图中，存在道路可以与其相连；其次，这些对象是无用的，及程序以后不会再使用这些对象。如果对象满足这两个条件，这些对象就可以判定为Java中的内存泄漏，这些对象不会被GC回收，然而它却占用内存。



## Android中内存泄漏的原因

在Android中内存泄漏的原因其实和在Java中是一样的，及某个对象已经不需要再用了，但是他却没有被系统所回收，一直在内存中占用着空间，而导致它无法被回收的原因大多是由于它被一个生命周期更长的对象所引用。对于Android内存泄漏的原因，我们总结起来就是一句话。**生命周期较长的对象持有生命较短的对象的引用。简单来说，避免内存泄漏的宗旨就是，生命周期长的不要和生命周期短的玩。**



**接下来我们分析一下内存泄漏的根本原因，下面就针对一些常见的内存泄漏进行分析。**

### 单例造成的内存泄漏

```java
public class Test {
    public static Test test;
    private Context context;

    public Test(Context context) {
        this.context = context;
    }

    public static Test getInstance(Context context) {
        if (test==null){
            synchronized (Test.class){
                if (test==null){
                    test=new Test(context);
                }
            }
        }
        return test;
    }
}
```

这是单例模式饿汉式的双重效验锁的写法，这里的 test持有了 Context对象，因为其是静态，如果getActivity中调用getInstance并传入当前context(非getApplicationContext),那么当Activity销毁时，Activity就无法回收，造成内存泄漏。所以我们修改其构造方法即可。

```java
private Test(Context context) {
    this.context = context.getApplicationContext();
}
```

通过context获取ApplicationContext，让它被单例持有，这样退出Activity时，Activity对象就能正常被回收了，而Application的Context的生命周期和单例的生命周期是一致的，所以在整个App运行过程中都不会造成内存泄漏。



### 非静态内部类造成的内存泄漏

我们知道，非静态内部类会持有外部类的引用，如果这个非静态的内部类的生命周期比它的外部类的生命周期长，那么当销毁外部类时，它无法被回收，就会造成内存泄漏。

**外部类中持有非静态内部类的静态对象**

```java
public class MainActivity extends AppCompatActivity {
    private static Test test;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if (test==null){
            test=new Test();
        }
    }
    private class  Test{
        
    }
}
```

这样的话，就导致非静态内部类Test持有外部类MainActivity的引用，导致MainActivity退出时不能被回收，从而造成内存泄漏。解决的办法就是把test改为非静态，这样test的生命周期和MainActivity就是一样的了，就避免了内存泄漏。或者把Test改成静态内部类，让test不持有MainActivity的引用。



### Handler或Runnable作为非静态内部类

Handler和 Runable 都有定时器的功能，当他们作为非静态内部类的时候，同样会持有外部类的引用，如果他们的内部有延迟操作，在延迟操作还没有发生的时候，销毁了外部类，那么外部类对象无法回收，从而造成内存泄漏：

```jsvs
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                
            }
        },10000);
    }
}
```

上面的代码中，Handler和Runnable作为匿名内部类，都会持有MainActivity的引用，而它们内部有一个10秒钟的定时器，如果在打开MainActivity的10秒内关闭了MainActivity，那么由于Handler和Runnable的生命周期比MainActivity长，会导致MainActivity无法被回收，从而造成内存泄漏。

**解决方案：**

一般套路就是把Handler和Runnable定义为静态内部类，这样它们就不再持有MainAcitivty的引用了，从而避免了内存泄漏。

```java
public class MainActivity extends AppCompatActivity {

    private TestHanlder testHanlder;
    private TestRunnable runnable;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        testHanlder=new TestHanlder();
        runnable=new TestRunnable();
        testHanlder.postDelayed(runnable,1000*10);
    }

    private static class TestHanlder extends Handler{

    }

    private static class TestRunnable implements Runnable{

        @Override
        public void run() {

        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (testHanlder != null) {
            testHanlder.removeCallbacks(runnable);
        }
    }
}
```



但是还是有一种特殊情况，如果Handler或者Runable中持有Context对象，那么即使使用静态内部类，还是会发生内存泄漏。

```java
public class MainActivity extends AppCompatActivity {

    private TestHanlder testHanlder;
    private TestRunnable runnable;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        testHanlder=new TestHanlder(this);
        runnable=new TestRunnable();
        testHanlder.postDelayed(runnable,1000*10);
    }

    private static class TestHanlder extends Handler{
        private Context context;

        public TestHanlder(Context context) {
            this.context = context;
        }
    }

    private static class TestRunnable implements Runnable{

        @Override
        public void run() {

        }
    }
}
```

这个时候可以采用弱引用来处理；

```java
public class MainActivity extends AppCompatActivity {

    private TestHanlder testHanlder;
    private TestRunnable runnable;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        testHanlder=new TestHanlder(new WeakReference<Context>(this));
        runnable=new TestRunnable();
        testHanlder.postDelayed(runnable,1000*10);
    }

    private static class TestHanlder extends Handler{
        private Context context;
        TestHanlder(WeakReference<Context> reference){
            this.context=reference.get();
        }
    }

    private static class TestRunnable implements Runnable{

        @Override
        public void run() {

        }
    }
}
```

通过以上的例子我们就可以明白，避免内存泄漏的宗旨就是，生命周期长的不要和生命周期短的玩。

其他内存泄漏例子还很多，比如ThrealLoal中的ThreadLocalMap中的key弱应用，MVP中的P层持有view未释放等等。