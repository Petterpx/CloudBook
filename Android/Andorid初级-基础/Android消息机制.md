# Android消息机制分析



##### .消息机制的模型

消息机制主要包含：MessageQueue，Handler和Looper这三大部分，以及Message，下面我们一一介绍。

**Message：**需要传递的消息，可以传递数据；

**MessageQueue：**消息队列，但是它的内部实现并不是用的队列，实际上是通过一个单链表的数据结构来维护消息列表，因为单链表在插入和删除上比较有优势。主要功能向消息池投递消息(MessageQueue.enqueueMessage)和取走消息池的消息(MessageQueue.next)；

**Handler：**消息辅助类，主要功能向消息池发送各种消息事件(Handler.sendMessage)和处理相应消息事件(Handler.handleMessage)；

**Looper：**不断循环执行(Looper.loop)，从MessageQueue中读取消息，按分发机制将消息分发给目标处理者。



![img](http://upload-images.jianshu.io/upload_images/3985563-6c25004471646c1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)



暂时问题就这些吧，我们现在开始，go

我们先在子线程创建一个Handler对象，然后调用post方法弹出一个Toast，看看会发生什么

```java
public class HandlerActivity extends AppCompatActivity {

    private Handler handler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler);
        new Thread(new Runnable() {
            @Override
            public void run() {
                Looper.prepare();
                handler = new Handler(Looper.myLooper());
                Looper.loop();
            }
        }).start();
        try {
            Thread.sleep(1000);
            handler.post(new Runnable() {
                @Override
                public void run() {
                    Toast.makeText(HandlerActivity.this, "123", Toast.LENGTH_SHORT).show();     
                }
            });
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

```

![1561769112752](../../../assets/1561769112752.png)

虽然报错了，但没关系，我们先梳理一下流程

### Handler流程

我们先看创建Handler对象后，内部做了些什么？

```java
public Handler() {
    //调用另一个构造函数
    this(null, false);
}
```

```java
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }
	//这里必须先执行Looper.prepare()，才能获取Looper对象，否则为null
    //从当前线程中获取Looper对象
    mLooper = Looper.myLooper();
    //如果looper为空，则抛出异常
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
    //消息队列
    mQueue = mLooper.mQueue;
    //回调方法
    mCallback = callback;
    //设置消息是否是异步处理方式
    mAsynchronous = async;
}
```

刚才我们子线程创建的Handler是一个无参构造方法，默认采用当前线程的Looper对象，并且回调方法为null,消息为同步处理方式。所以当我们直接运行之后，因为没有Looper对象，所以编译器直接报错。

但为什么主线程运行的却好着呢？我们看一下源码

```java
//程序在启动的时候，系统以及帮我们自动调用了Looper.prepare()方法
public static void main(String[] args) {
  	。。。
   Looper.prepareMainLooper();
 	 。。。
   Looper.loop();
}

->我们看主线程调用的prepareMainLooper()方法
public static void prepareMainLooper() {
    //false表示这个Looper无法退出
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        //从ThreadLocal中获取Looper对象
        sMainLooper = myLooper();
    }
}

->然后再看prepare方法，接着看 Looper(quitAllowed),ThreadLocal.set我们一会再看
private static void prepare(boolean quitAllowed) {
     if (sThreadLocal.get() != null) {
         throw new RuntimeException("Only one Looper may be created per thread");
     }
     //将looer对象存放在线程本地存储区中，简称TLS
     sThreadLocal.set(new Looper(quitAllowed));
 }
 
->初始化Looper
   private Looper(boolean quitAllowed) {
   		//创建消息队列
        mQueue = new MessageQueue(quitAllowed);
        //保存本地线程对象
        mThread = Thread.currentThread();
    }

```

通过对上面方法的分析，我们可以知道，在ui线程中创建Handler时，由于ActivityThread已经调用了Looper.prepare方法，所以已经有了Looper对象，我们直接可以通过post，或sendMessage()方法，发送消息，然后由Looper取出消息，再分发给Handler进行处理，关于这个过程，我们来慢慢分析。



### Looper

我们上面刚才频繁说到了Looper，那么我们就来分析一下Looper.的一些方法。

初始化Looper的流程我们刚在上面已经说过了，这里我们主要说一下它的另一个方法 Looper.loop()，也就是开启Looper

```java
public static void loop() {
    //获取ThreadLocal中的Looper对象
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    //获取Looper对象中的消息队列
    final MessageQueue queue = me.mQueue;

    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();
    final int thresholdOverride =
            SystemProperties.getInt("log.looper."
                    + Process.myUid() + "."
                    + Thread.currentThread().getName()
                    + ".slow", 0);

    boolean slowDeliveryDetected = false;
	
    //进入loop的主循环方法，这也就是为什么它可以一直取消息的原因
    for (;;) {
        //可能会阻塞，因为next()方法可能会无限循环
        Message msg = queue.next(); // might block
        //如果消息为null，则退出循环
        if (msg == null) {
            return;
        }

  		//默认为null,可以通过setMessageLogging()方法来指定输出
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
        long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
        if (thresholdOverride > 0) {
            slowDispatchThresholdMs = thresholdOverride;
            slowDeliveryThresholdMs = thresholdOverride;
        }
        final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
        final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

        final boolean needStartTime = logSlowDelivery || logSlowDispatch;
        final boolean needEndTime = logSlowDispatch;

        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }

        final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
        final long dispatchEnd;
        try {
            //获取msg的handler对象，用于分发消息
            msg.target.dispatchMessage(msg);
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        if (logSlowDelivery) {
            if (slowDeliveryDetected) {
                if ((dispatchStart - msg.when) <= 10) {
                    Slog.w(TAG, "Drained");
                    slowDeliveryDetected = false;
                }
            } else {
                if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                        msg)) {
                    // Once we write a slow delivery log, suppress until the queue drains.
                    slowDeliveryDetected = true;
                }
            }
        }
        if (logSlowDispatch) {
            showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }
        msg.recycleUnchecked();
    }
}
```

以上就是Looper的工作原理，简单来说就是它会不停的从MessageQueue中通过next()查看是否有新消息，即是一个死循环，唯一跳出循环的方法就是MessageQueue的 next()方法返回了null,才可以跳出循环。

当然Looper还有一些别的方法。

比如像 getMainLooper，可以用来获取looper对象。

```java
public static Looper getMainLooper() {
    synchronized (Looper.class) {
        return sMainLooper;
    }
}
```

或者说使用quit,和quitSafely来退出一个Looper。二者的区别是，quit会直接退出Looper，而quitSafely只是设定了一个标记，然后把消息队列中的已有消息处理完才会退出Looper.



### ThreadLocal

**然后我们来看一下ThreadLocal，ThreadLocal是什么？**

> 翻译过来就是本地线程组，那么什么是本地线程组呢。
>
> ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到。
>
> 我们通过源码来看

我们先看ThreadLocal 的set方法

```java
public void set(T value) {
    //获取线程对象,map的键
    Thread t = Thread.currentThread()；
    //ThreaLocalMap 划重点
    ThreadLocalMap map = getMap(t);
    if (map != null)
        //存入相应的value
        map.set(this, value);
    else
        //如果是第一次，先创建映射
        createMap(t, value);
}
->
    //初始化ThreadLocalMap
 void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
	}
->
 ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    		//创建初始容量为16的数组
            table = new Entry[INITIAL_CAPACITY];
    		//通过键，计算hash值，相当于计算下标
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    		//添加value
            table[i] = new Entry(firstKey, firstValue);
    		//声明map长度为1
            size = 1;
    		//调整负载因子
            setThreshold(INITIAL_CAPACITY);
        }
```

是不是发现了个ThreadLocalMap的东西，这是个静态内部类啊，那么它是用来干啥的？

> 从注释上看一下，它是ThreadLocal的一个静态内部类。用来维护线程私有值创建的自定义

我们接着来看看它的具体解释。

源码是最好的老师，这句话嗯，没错，妈卖批

```java
static class ThreadLocalMap {
    	//存储数据为Entry,其中key是弱引用
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    	//初始大小
        private static final int INITIAL_CAPACITY = 16;
        private Entry[] table;
    	//map长度
        private int size = 0;
    	//负载因子，用于数组容量的扩容
        private int threshold; // Default to 0
    	//调整负载因子，默认为当前数组长度2/3
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }
    	//这里我们上面说过了，第一次放入Entry数据时执行的操作
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```

从这里我们可以看出，在第一放入数据时，会初始化数组长度为16，然后定义数组的扩容阈值为当前默认的2/3



上面我们在ThreadLocal 的set方法里见到了，map.set()方法，我们接着来看看它都做了哪些操作

```java
private void set(ThreadLocal<?> key, Object value) {
    
   
    Entry[] tab = table;
    int len = tab.length;
    //根据hash值计算位置
    int i = key.threadLocalHashCode & (len-1);
    
    //判断当前位置是否有数据，如果key相同，就进行替换，否则插入新数据
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //key值相同，直接覆盖
        if (k == key) {
            e.value = value;
            return;
        }
        //key值为null，清空所有key为null的数组
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //上面两种都不满足，直接添加
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //判断是否需要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```



我们需要的就这些，详细的步骤，可以看这篇博客<https://www.jianshu.com/p/2a34d30806d4>

好了说了这么一堆，就算是打点基础了，现在继续沿着我们的思路走。

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //将looper存入ThradLocalMap，当前线程对象当做键
    sThreadLocal.set(new Looper(quitAllowed));
}
```

我们已经知道了这里是存入map，而且每一个线程都是独有的键值对。现在来看new Looper(quitAllowed) 时，都干了什么？在看之前，我们先了解一点Looper的基础知识。



### MessageQueue

#### 我们先来看一下它的MessageQueue的工作原理

先从它的post方法开始，内部调用的是sendMessageDelayed()，最终调用的也是sendMessageAtTime();

```java
public final boolean post(Runnable r)
{	
    //将消息添加到消息队列中， getPostMessage
   return  sendMessageDelayed(getPostMessage(r), 0);
}

->
   public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

```

我们主要分析一下这个方法

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    //消息队列，从Looper中获取的
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }	
    //调用enqueMessage
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    //调用MessageQueue的enqueueMessage
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

可以看到sendMessgaeAtime()方法的作用就是调用MessageQueue的enqueeuMessage()方法，往消息队列中添加一个消息。下面我们具体来看enqueueMessage的操作。



boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

```java
 boolean enqueueMessage(Message msg, long when) {
     	//每一个Message必须都有一个target
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            //退出时，回收msg，加入消息池
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
           	//如果MessageQueue没有消息，或者msg触发时间是队列中最早的，则进入该分支
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
               	//将消息按时间顺序插入到MessageQueue
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
enqueueMessage是按照Message触发时间先后顺序排列的，队头的消息是将要最早触发的消息，当有消息需要加入消息队列时，会从队列头开始遍历，直到找到消息应该插入的合适位置，以保证所有消息的时间顺序。说简单点就是单链表的插入操作。

我们接下里看看刚才在Looper中用过的 next()方法，也就是用来获取消息的方法。

```java
Message next() {
    final long ptr = mPtr;
    //当消息循环已经退出，则直接返回
    if (ptr == 0) {
        return null;
    }

    //循环迭代的首次为-1
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        //阻塞操作，当等待nextPollTimeoutMillis时长，或者消息队列被唤醒时，都会返回
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            //当消息Handler为空时，查询MessageQueue中的下一条异步消息msg，为空则退出循环
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    //当异步消息触发时间大于当前时间，则设置下一次轮训的超时时长
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    //设置消息的状态
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
             
                nextPollTimeoutMillis = -1;
            }
			//消息正在退出，返回null
            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }
            。。。
    }
}
```

nativePollOnce是阻塞操作，其中nextPollTimeoutMillis代表下一个消息到来前，还需要等待的时长；当nextPollTimeoutMillis = -1时，表示消息队列中无消息，会一直等待下去。
可以看出`next()`方法根据消息的触发时间，获取下一条需要执行的消息,队列中消息为空时，则会进行阻塞操作。

在loop方法中，获取到下一条消息后，执行msg.target.dispatchMessage(msg),来分发消息到目标Handler对象。我们接下来来看看它的具体操作

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        //当Message存在回调方法，回调msg.callback.run()方法
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler存在Callback成员变量时，回调方法handleMessage（）
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //handler自身回调方法handleMessage()
        handleMessage(msg);
    }
}

->
        private static void handleCallback(Message message) {
        message.callback.run();
    }
```

**分发消息流程：**
当Message的`msg.callback`不为空时，则回调方法`msg.callback.run()`;
当Handler的`mCallback`不为空时，则回调方法`mCallback.handleMessage(msg)`；
最后调用Handler自身的回调方法`handleMessage()`，该方法默认为空，Handler子类通过覆写该方法来完成具体的逻辑.

**消息分发的优先级：**
Message的回调方法：`message.callback.run()`，优先级最高；
Handler中Callback的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；
Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。

对于很多情况下，消息分发后的处理方法是第3种情况，即`Handler.handleMessage()`，一般地往往通过覆写该方法从而实现自己的业务逻辑。

![img](http://upload-images.jianshu.io/upload_images/3985563-b3295b67a2b0477f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)



**回顾一下消息机制的运行流程：**在子线程执行完耗时操作后，当Hadnler发送消息时，将会调用MessageQueue.enqueMessage向队列中添加消息。当通过Looper.loop 开启循环后，会不断从队列中读取消息，即调用 MessageQueue.next, 然后调用目标Handler（即发送消息的Handler）的dispatchMessage传递消息，然后返回到Handler所在线程，目标Handler收到消息后，调用handleMessage方法，接受消息，处理消息。

日常使用时，需要注意的是，因为ThreadLocal中的Entry[]数组的键是弱引用，所以我们再使用

时要么将Handler设为静态，要么使用完handler之后，调用相应的remove方法，关闭looper等。

**为什么ThreadLocal可以做到不同线程中取到不同的值**：是因为在ThreadLocal中有一个ThreadLocalMap的静态内部类，它的作用就是用来存储不同线程的不同value值，当每次set，或get的时候，它都会先获取当前线程的对象，然后将当前对象当做key，计算出它的hash值也就是下标，然后再来进行操作。





Looper.loop（）为什么不会死循环卡死？

> 简单一句话是：Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。
>
> \---
> 这一题是需要从消息循环、消息发送和消息处理三个部分理解Android应用程序的消息处理机制了，这里我对一些要点作一个总结：
>
> ​         A. Android应用程序的消息处理机制由消息循环、消息发送和消息处理三个部分组成的。
>
> ​         B. Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。
>
> ​         C. Android应用程序的主线程进入空闲等待状态的方式实际上就是在管道的读端等待管道中有新的内容可读，具体来说就是是通过Linux系统的Epoll机制中的epoll_wait函数进行的。
>
> ​         D. 当往Android应用程序的消息队列中加入新的消息时，会同时往管道中的写端写入内容，通过这种方式就可以唤醒正在等待消息到来的应用程序主线程。
>
> ​         E. 当应用程序主线程在进入空闲等待前，会认为当前线程处理空闲状态，于是就会调用那些已经注册了的IdleHandler接口，使得应用程序有机会在空闲的时候处理一些事情。

<https://www.jianshu.com/p/20f10e50f3ec>  Android Handler机制 - handleMessage究竟在哪个线程执行

<https://www.jianshu.com/p/02962454adf7>Android 消息处理机制（Looper、Handler、MessageQueue,Message）

