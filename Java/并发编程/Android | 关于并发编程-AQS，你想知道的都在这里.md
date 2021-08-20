# Android | 关于并发编程-AQS，你想知道的都在这里

## 引言

如果问一个 Android 同学，请你简单说一下 Java AQS 的基本思想，那么有不少于一半的同学可能是懵逼状态😨。

> 什么玩意，AQS 是什么，我咋没听过🙃。

的确，对于非Java后端同学来说，没听过倒也不是什么太过分的事，但是如果你深入学习过 Java 并发相关，那么肯定会去了解各种锁,而作为一个 **有志青年** 的你必然会在心里来一句，**为什么加了锁就可以同步** ？ 此时必然也会看到 `AQS` 的影子。

从技术的角度讲，当我们谈到 `ReentrantLock` ，不难也会说到 `AQS` 。

如果上述所讲你不了解的话，那么本篇文章可能会对你有所帮助。

## 背景

在了解一项技术之前，我们有必要了解这项技术所产生的原因，及它能为我们带来什么？

`AQS` 全名 `AbstractQueuedSynchronizer`，在 **jdk1.5** 时加入 ，是整个 `JUC` 的基础，我们现在用到的大多数同步器都是基于 `AQS` 的，其性能相比于 常用的 `synchronized` 在一定程度上提高了不少，而它的构建者 也就是并发包的大师**（Doug Lea）** 更是期望它能够成为实现大部分同步需求的基础。

**那AQS出现原因是什么呢？** 难道 `AQS` 的出现仅仅是为了提升性能吗，或者说仅仅是因为性能，就要重复造一个 `AQS` 的轮子？

要解释这个问题，首先我们要思考一下在它出现之前，我们经常使用的同步方式是 `synchronized` ，而 `synchronized` 是没办法解决死锁，因为其在申请资源时，如果申请不到，线程将直接进入阻塞状态，也即释放不了线程已经占用的资源，而为了解决这个问题，我们就需要新的方案。

如果让我们自己重写去设计一个工具来解决上述问题，那该怎么去设计，AQS提供了以下方案？

- **支持响应中断**。 `synchronized` 一旦进入阻塞状态，就无法被中断。但如果阻塞状态的线程能够响应中断信号，能够被唤醒。这样就破坏了不可被抢占条件了。
- **支持超时。** 如果线程在一段时间之内没有获取到锁，不是进入阻塞状态，而是返回一个错误，那这个线程也有机会释放曾经持有的锁，这样也能破坏不可抢占条件。
- **非阻塞的获取锁。** 如果尝试获取锁失败，并不进入阻塞状态，而是直接返回，那这个线程也有机会释放曾经持有的锁。这样也能破坏不可抢占条件。

> 上述背景内容来源(有修改) [binarylei](https://www.cnblogs.com/binarylei/p/12555166.html)



## AQS是什么？

抽象队列同步器 `AbstractQueuedSynchronizer` ，**是用来众多同步组件的一个 基础框架**  ， 内部用到了 `CLH` 队列锁 。

如果之前没了解过，那么这句话可能听着有点懵，先别急着搞清原理，我们先在脑海里有一个概念 ：

**这玩意是用来构建锁的基础框架** , **这玩意是用来构建锁的基础框架** , **这玩意是用来构建锁的基础框架**

> 有人说，AQS 只是一个工具类而已，的确如此。但是其的重要性在并发编程中用 **框架** 这个词或许更为合适。



## 1. AQS 基础理论入门

`AQS` 其主要使用方式是继承，即子类通过继承 `AQS` 实现它的抽象方法来管理状态，内部使用一个 int 成员变量 `state` 表示同步状态,并且通过一个 `FIFO(先进先出)` 的队列来完成线程的排队工作。

在具体的实现上，通常子类推荐被定义为静态内部类(就像 ReentrantLock中 的 `Sync` )，`AQS` 本身没有实现任何同步接口，它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用，同步器既可以支持独占式的获取同步状态，也可以支持共享式的获取同步状态，这也是 jdk 中内置的同步组件的实现原理。

### 总览

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gpphepro41j31ls0u07ap.jpg" alt="image-20210419234207972" style="zoom:67%;" />

描述一下工作流程：

每次在请求一个资源时，先将其添加到 `CLH` 队列中，并且 **state+1** , 队列处于循环遍历阶段(当 **state=0** 时，代表资源无占用)，队列循环过程中，如果被请求的共享资源空闲状态，则将当前请求资源的线程设置为有效的工作线程，并且更改其node节点中的 `locked` 为 `true` ,代表其处于占用状态，后续结点(即后续需要获取资源的线程)就一直处于自旋等待状态，不停的判断前一个节点 的 `locaked` 是否为 `false` ,如果为 `false` ,则证明此时资源处于空闲状态，则去尝试获取资源。

### 模板方法模式

如果翻阅 `ReentrantLock` 的源码，会发现一个问题，那就是 除了最开始的 `lock()` 是 `ReentrantLock` 本身提供的方法之外,而剩下的方法解析，也就是是其内部的核心方法调用，都是调用了 `AQS` 的相关方法。

并且都 `NonfairSync` 实现了 `AQS` 的一些方法，比如 `tryAcquire()` ,以便于提供具体的实现逻辑。

而这样的设计我们称之为 **模板方法模式** ：

即也可以总结为：对于一个设计而言，我们将不容易改变的方法提前定义好具体的实现逻辑，并且将一些需要具体实现的交给外部自己实现。并将这种类似的方法按照我们的逻辑需要，包装成几个通用的方法调用合体，以供外部调用。

- 对于外部而言，其本身并不知道我内部的具体实现；
- 外部需要实现自己相应的独特逻辑的几个特定方法；

这样的好处就是 我们封装了不变的部分，仅扩展了可变部分，并且提取了公共代码，以便于维护。



### CLH队列锁

**CLH队列锁** 是一种基于链表的可扩展，高性能，公平的自旋锁，申请资源的线程仅仅在本地变量上自旋，它不断轮训前驱节点的状态，假设发现前驱释放了锁，就结束自旋。

当一个线程需要获取锁时，先创建一个的 `QNode` ,将其中的 `locked` 设置 **true** 表示需要获取锁， `myPred` 表示对其前驱结点的引用。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gqr7rbk7o9j30m00bwjrv.jpg" alt="image-20210522145956491" style="zoom: 50%;" />



其完整流程如下所示：

1. 假设线程A要获取资源，其先使自己成为队列的尾部，同时获取一个指向其前驱结点的引用 `myPred`，并不断在父节点引用上自旋判断。

   <img src="https://tva1.sinaimg.cn/large/008i3skNly1gtn2mnzchuj60s80gc3zf02.jpg" alt="image-20210820110114023" style="zoom:50%;" />

2. 当另一个线程B同样也需要获取锁时，上述的过程同样也要来一遍，如下所示 **(QNode-B)**：

   ![image-20210820110135569](https://tva1.sinaimg.cn/large/008i3skNly1gtn2n1ssmhj619e0i0ach02.jpg)

3. 当某个线程要释放锁时，就将当前节点的 `locked` 设置为 `false` 。

   其后续节点因为不断在自旋，当判断到其前序节点 `locked` 为 `false` ,就表明其前序节点已经释放锁，其自身就可以获取到锁，并且释放当前前序节点引用，以便GC回收。

   ![image-20210820110617378](https://tva1.sinaimg.cn/large/008i3skNly1gtn2rxhs2wj61di0i4wgv02.jpg)



整个过程如上图所示，**CLH队列锁** 的优点是空间复杂度低,如果有n个线程，L个锁，每个线程每次都只获取一个锁，那么其需要的存储空间 **O(L+n)** ,n个线程有n个 node,L 个锁有L个tail .

`AQS` 中的 **CLH队列锁** 实现方式与上述方式相比是一种变体的实现，相比普通 **CLH队列锁** ，`AQS` 中的实现方式 做了相关的优化，比如不会不断重试，而会在重试相关次数后将线程阻塞。等待之后的唤醒。

### AQS 相关方法

#### 模板方法

在我们实现自定义的同步组件时，将会调用同步器提供的模板方法。相关方法如下：

![image-20210522153916558](https://tva1.sinaimg.cn/large/008i3skNly1gqr8w78zexj317a0q2qle.jpg)

上述模板方法同步器提供的模板方法分为3类：

1. 独占式获取与释放同步状态
2. 共享式获取与释放
3. 同步状态和查询同步队列中的等待线程情况。

#### 可重写的方法

![image-20210522154257997](https://tva1.sinaimg.cn/large/008i3skNly1gqr901dumjj314c0ecahi.jpg)

#### 访问或修改同步状态的方法

在自定义的同步组件框架中，`AQS` 抽象方法在实现过程中免不了要对同步状态 `state` 进行更改，这时就需要同步器提供的3个方法来进行操作，因为他们能够保证状态的改变是安全的：

- getState()  

  获取当前同步状态

- setState(newState:Int) 

  设置当前同步状态

- compareAndSetState(expect:Int,update:Int)

  使用 CAS 设置当前状态，该方法能保证状态设置的原子性。





## 2. 从ReentrantLock到AQS

没看过AQS，**ReentrantLock** 总该了解点吧，有道是知兄莫如弟，那么我们就由其入手，旁敲侧击。

### 简述

描述一下ReentrantLock的背景：

> 我们都知道  `synchronized` 关键字是用于加锁，但是这种锁对于性能影响比较大，因为线程在获取资源时必须处于等待状态，没有额外的尝试机制。所以在jdk1.5 的时候，java 提供了 `ReentrantLock` ，用于替代  `synchronized`.
>
> `ReentrantLoack` 具有可重入，可中断，可限时，公平锁非公平锁等特点。

ReentrantLock 的简单使用：

```kotlin
val lock = ReentrantLock()
lock.lock() //加锁
//业务逻辑
lock.unlock()  //释放 
```



### 与AQS的关系

看到这，你可能会说，你说了那么多，那它到底和 `AQS` 有啥关系？请看下图所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gqrarm1tc5j318z0u0q6f.jpg" alt="image-20210422214527370" style="float: left;" />

`ReentrantLock` 内部有一个抽象内部类 `Sync` 继承了 `AbstractQueuedSynchronizer` ，默认构造函数中又实例化了 `NonfairSync` 类  (`Sync` 的子类)。对于外部而言，只关注 **lock** 与 **unlock** 方法，但实际上内部都是调用了 `AbstractQueuedSynchronizer` 的方法。



### 流程剖析

看了上面的图，只是为加深一个印象，那就是 `ReentrantLock` 中用到了 `AQS` ,接下来我们通过下面这个简单流程解析，来看一下 AQS 在 `ReentrantLock` 的运用以及其原理。

为了便于理解，我们整个流程都是以 `NonfairSync ` 即不公平锁的伪源码为例(公平非公平差距并不大)。

#### lock()

从加锁方法开始，如下：

```kotlin
class NonfairSync .. -> 

   fun lock() -> {
  
      👉 1. if (compareAndSetState(0, 1))
            👉 2. setExclusiveOwnerThread(Thread.currentThread())
            else
               👇 acquire(1) //独占模式获取	
 
       
       fun acquire(arg:Int=1) -> {
				  //尝试获取 && 将当前线程添加到等待队列中，并通过CAS的方式不断尝试获取前一个节点
         👉 3.  if (!tryAcquire(arg) &&
             👉 4.  acquireQueued(addWaiter(Node.EXCLUSIVE), arg) )
         
```

##### 1.compareAndSetState

这个方法的意思是尝试获取锁，其内部的操作如下：如果当前状态值等于期望值，则以原子方式将同步状态设置为给定的更新值。

也就是说，当前我们预估值为0，即我们预估当前没有线程占用资源，如果操作时，发现 这个要实际操作的值真的是0，也就是当前资源并没有其他线程占用，那么我们就将其更新为1，表示当前资源已经被占用。

而 `AQS` 内部正是有一个 `int` 型变量 `state`  ,其作用正是代表当前加锁状态。

```java
private volatile int state;
```

当线程尝试获取锁成功后，如果同一个线程再次尝试获取锁呢？我们称之为锁的重入，那怎么做呢？总不能我自己再获取一把锁？不可能吧，对于一个资源，怎么可能生成两把锁被同一个线程占用。离谱！那怎么办呢？

这时候就轮到 `setExclusiveOwnerThread` 方法了，我们看看它的实现。

##### 2.setExclusiveOwnerThread

```kotlin
protected final void setExclusiveOwnerThread(Thread thread) - {
        exclusiveOwnerThread = thread
```

内部是设置了当前的线程对象，而这个 `exclusiveOwnerThread` 正是 `AQS` 另一个变量，代表了 **当前拥有锁的线程** 。这个在哪里用呢，我们看下面方法。

##### 3.tryAcquire

这个方法的含义是以不公平的方式去获取锁，其伪代码如下：

```kotlin
 
 fun tryAcquire(acquires:Int=1) -> {
    👇
     fun nonfairTryAcquire(acquires):Boolean -> {
  
          val current = 当前线程对象
    	    val c = getState()
     1. 👉 if(c== 0 && compareAndSetState(0, acquires)) 
    	       setExclusiveOwnerThread(current)
    		     return true
    	 
     2. 👉 else if (current= AQS中当前占用资源的线程对象)
    	       AQS中持有的state += acquires
             return true
    	
           return	false

```

> 当调用 **nonfairTryAcquire** 获取锁时,内部的操作很简单：首先获取当前的线程对象与 当前 `AQS` 中储存的 `state` 状态值，
>
> 1. 如果当前state=0 并且 通过 **compareAndSetState** 方法尝试修改 `state` 成功 则代表当前资源没有线程占用，然后就设置当前拥有锁的线程为当前自己。
> 2. 如果 当前占用资源的线程是自己，那么对 `AQS` 中的 **state+1** ,然后返回true，即代表当前线程获取锁成功。
>
> 如果 return **false**,则代表当前线程获取锁失败。

为什么这里当获取锁的时候是同一个线程就要 `state+1` 呢？

我们都知道，使用 `ReentrantLock` 时，我们释放锁调用的是 **unLock** ,那么我们的切入点就在这了。

##### 4.acquireQueued

这个方法是以死循环的方式不断获取锁，内部代码如下：

```kotlin
 fun acquireQueued(node:Node, arg:Int=1):Boolean  -> {
    //当前是否成功拿到资源
    var failed = true
    try {
      	//是否在等待过程中被中断过
        val interrupted = false
      	//自旋开始，要么获取锁，
        while (true) {
            val p = 前一个node节点
          	//如果前一个节点是head节点并且修改state成功，则表明当前线程已获得锁
            if (p == head && tryAcquire(arg)) {
              	//设置新的头结点
                setHead(node)
              	//将之前的头结点置null,便于GC回收
                p.next = null // help GC
                failed = false
                return interrupted
            }
          	//获取锁失败时调用
          	//如果通过前驱结点判断发现当前线程被阻塞并且当前线程已经被中断，则修改 interrupted 标记
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true
        }
    } finally {
        if (failed)
            cancelAcquire(node)
    }
}
```

##### 总结

当我们调用 `lock()` 方式加锁时(以不公平锁为例)，内部先是以原子方式去尝试修改 `AQS` 中持有的 **state** 变量：

1. 如果修改成功，则代表当前资源无线程占用，则获取锁成功，并且将当前 `AQS` 中的 `exclusiveOwnerThread` 更新为当前线程对象，以便于后期锁重入时直接返回获取锁成功。
2. 如果最开始修改失败，则调用 `acquire` 方法去获取锁。 其方法会内部 尝试获取锁一次 ，并且将当前线程添加到等待队列中，然后通过CAS的方式不断自旋，一直获取 **父node** 节点, 如果 **父node** 节点是 **head头节点** ，就说明当前节点在 **队列首部 ** ，就尝试获取锁，如果获取成功，则更新队列头节点为当前node,并移除当前遍历的父节点。如果获取锁失败，则通过前驱节点判断当前线程是否被阻塞，如果当前线程已经被中断，则更新标记位，并且暂停此循环，等待唤醒。

看完了 **lock()** 方法的简单分析，是不是觉得感觉自己上错了车，上面只是简单做了一个流程分析，如果细追下去，其中的细节还很深，可能就不是本文所能全部概述，我们只需要知道大体流程即可。

#### unLock()

```kotlin
   1.👇
   fun unLock() -> { 
  
       2.👇
       fun release(arg:Int=1):Boolean -> {
	 		 						// 如果 -tryRelease- 结果为true,则唤醒正在等待的队列，即让其他线程获取锁。  
      	     👉   if (tryRelease(arg))
            		    ...
          			   unparkSuccessor(h)
      
           3.👇
           fun tryRelease(arg=1):Boolean -> {
             
                c = AQS中state变量 - arg
                ..
                if (c == 0) { 
                    //设置AQS中持有的线程为null
                    setExclusiveOwnerThread(null)  
             				return true 
                }
             		..
                setState(c)
             		..       
             
```

如上所述伪代码， **unLock ** 方法调用顺序如下，在调用 **unLock** 方法进行释放锁时，内部其实调用了 **relase** 方法，其内部又调用了 **tryRelease** 方法,其内部先是使用 `AQS` 中的 **state 变量-arg(1)** ，如果当 **c=0** ，则表明当前已经没有线程占用资源，则去唤醒正在等待中的队列，也就是让其他线程开始获取锁。



#### 串一遍思路(非公平锁)

当我们调用 **lock** 方法时，先是尝试以原子的方式去修改 `AQS` 内部的state变量值，如果当前 `state` 值与预期值一致，则更新 `AQS` 内部`state` 的变量值为 **1** ，并将当前线程对象的引用赋值给 `AQS` 。

如果在尝试修改 `state` 变量值的时候失败了，则调用  **acquire(xx)** 去获取锁，在方法内部将自己添加到当前等待队列中，并且以 `CAS` 的操作不断自旋，不断尝试去获取当 **父node节点** 的前一个节点是否等于 **head节点** ，并且当前线程是否已经尝试拿到锁，如果前一个节点等于 **head节点** 并且当前修改 `state` 变量成功，则代表当前线程已经拿到锁，则将 **当前node** 节点置为头结点，并移除其前一个节点。当然，`AQS` 对这个做了很多处理，它并不会一直重复上述重试操作，当经历一段自旋后，它就会以线程中断的方式停止下来，并且取消当前的尝试。

通过理一遍 `ReentrantLock` 的源码，我们大致了解了一下整个流程，及相应方法的具体职责，这对我们理解 `AQS` 将起到一些重要的作用。以及自定义一个 自己的重入锁 也将会有帮助。





## 3. 用AQS写一个重入锁

**锁的可重入**

> 指的是当某个线程调用某个方法或者对象获取了一把锁时，再次调用了指定方法，导致的锁的重入。即本身已经获取到了锁，又一次经历了锁的获取，一般情况下，我们会在再次进入时判断当前线程是否获取了锁，如果获取了，就修改同步状态，即 `AQS` 中的 **state+1** 。为什么要**state+1** ,因为释放锁的时候需要-1啊。

具体代码如下：

![image-20210523212913946](https://tva1.sinaimg.cn/large/008i3skNly1gqsomnldatj31fv0u0kha.jpg)





## 4. AQS于我们的日常

说实话，不会使用 `AQS` ，并不会影响开发任何，在Android开发的现在，各种线程相关的工具库，`Rx` , `协程` ，都是在降低开发难度，但作为基础，我们还是应该明白有些底层的设计思想，当你或许有一天想要自己去定义一个特定规则的线程工具时，这些看上去好像对我们实际用处不大的东西就都会派上用场。

任何东西的学习，都免不了一个 **为什么** ？

比如为什么加了 `synchronized` 就可以加锁，为什么 `ReentrantLock` 是可重入呢，当你想要搞清楚这些原因的时候，这些看起来晦涩的东西就是唯一入口。



## 插图地址

- [processon-AQS框架分析](https://www.processon.com/view/link/60aa5e697d9c08169188bfca)
- [processon-CLH队列锁分析](https://www.processon.com/view/link/60aa5f86f346fb77770ccd5d)

## 感谢

- [享学课堂 - AQS解析-Mark](https://ke.qq.com/webcourse/347420/102247978#taid=9160778695658780&vid=5285890803146885393)
- [锁原理 - AQS 源码分析：有了 synchronized 为什么还要重复造轮子](https://www.cnblogs.com/binarylei/p/12555166.html)

## 关于我

Hello，我是[Petterp](https://github.com/Petterpx),一个在帝都苟延残喘的Android工程师。

如果您觉得文章对您有价值，欢迎👏🏻，也欢迎关注我的 [Github](https://github.com/Petterpx) 。

如果您看完之后觉得差了点意思，也欢迎留言给我。

