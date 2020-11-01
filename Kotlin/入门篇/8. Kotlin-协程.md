# Kotlin-协程<*>

## 协程-入门

要点：**挂起，恢复**

### 协程是什么？

- 由程序自行控制 **挂起**，**恢复** 的程序
- 协程可以实现多任务的协作执行
- 协程可以用来解决异步任务控制流的灵活转移



### 协程的作用

- 协程可以让异步代码同步化
- 协程可以降低异步程序的设计复杂度
- 挂起和回复可以控制执行流程的转移
- 异步逻辑可以用同步代码的形式写出

***协程并不会提升性能,协程只是会提高代码的可读性***

### 协程与线程

协程 **Coroutine** : 指语言实现的协程，运行在内核线程上

线程**Thread**: 指操作系统的线程，也称为内核线程



### 关于协程你应该了解的：

- 它是Kotlin标准库
- 协程上下文
- 拦截器
- 挂起函数
- 官方协程框架
- job
- 调度器
- 作用域



## 协程的常见实现

### 协程的分类

#### 按调用栈

- 有栈协程：每个协程会分配单独的调用栈，类似线程的调用栈
- 无栈协程：不会分配单独的调用栈，挂起点状态通过闭包或对象保存(kotlin)

#### 按调用关系

- 对称协程：调度权可以转移给任意协程，协程之间是对等关系
- 非对称协程：调度权只能转移给调用自己的协程，协程存在父子关系

 



## 挂起函数 

- 以 **suspend** 修饰的函数
- 挂起函数只能在 **其他挂起函数** 或 **协程** 中调用
- 挂起函数调用时包含了协程 “挂起” 的语义
- 挂起函数返回时则包含了协程 ”恢复“ 的语义

***Kotlin内部通过 Continuuation 实现挂起函数***

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}

//成功
public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

//异常
public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))
```



### Continuation

***我们一直在说Continuation,那它到底在那里用了呢？*** 用一个例子来证明

#### 挂起函数 -案例

![image-20200108205803900](https://tva1.sinaimg.cn/large/006tNbRwly1gapg9t4znsj30jt02ddfv.jpg)

##### 转成字节码:

![image-20200108205644488](https://tva1.sinaimg.cn/large/006tNbRwly1gapg8fl903j30kl044mxj.jpg)

##### 调用处

> 当转成java字节码后，我们发现它需要 **Continuuation**这个参数。而Continuuation是编译器自动给我们传入的，只存在于挂起函数或者协程中。于是当我们在别的地方调用时，as就会提示报错。

**如果我们不在指定位置调用，则提示如下：**

![image-20200108205715032](https://tva1.sinaimg.cn/large/006tNbRwly1gapg8yod34j30j103hjrj.jpg)	



#### 将回调转写成挂起函数

##### 将Retrofit请求回调转为挂起函数

```kotlin
interface IBaidu {
    @GET("https://www.baidu.com/")
    fun getBaidu(): Call<String>
}
...
   suspend fun getBaidu() = suspendCoroutine<String> { continuation ->
        Re.retrofit.create(IBaidu::class.java)
            .getBaidu()
            .enqueue(object : Callback<String> {
                override fun onFailure(call: Call<String>, t: Throwable) {
                    continuation.resumeWithException(t)
                }

                override fun onResponse(call: Call<String>, response: Response<String>) {
                    response.takeIf {
                        it.isSuccessful
                    }?.body()?.let(continuation::resume)
                        ?: continuation.resumeWithException(HttpException(response))

                }
            })
    }
```

在上面的🌰中，我们知道，在调用网络请求方法 ***getBaidu***   时，系统已经为我们传入了 ***continuation***，那么应该如何使用呢？

> 我们通过调用 ***suspendCoroutine*** 方法来获得我们的 **Continuation** 接口，从而调用 ***resumeWithException***  实现函数挂起。

而一个函数的真正挂起需要实现异步的调用，在我们上面的🌰中，***enqueue*** 内部已经实现了 io 线程的切换，所以我们可以直接使用 **resumeWithException** 实现函数挂起.

所以 ***真正的挂起*** 必须异步调用 resume,包括以下：

1. 切换到其他线程 resume
2. 单线程事件循环异步执行

相反，如果我们直接 return 或者没有切线程则没有真正挂起。



##### 将回调转写为挂起函数：

- 使用 suspendCoroutine 获取挂起函数的 Continuation
- 回调成功的分支使用 Continuation.resume(value)
- 回调失败则使用 Continuation.resumeWithException(e)





### 协程的创建

- 协程是一段可执行的程序
- 协程的创建通常需要一个函数
- 协程的创建同时需要一个API

 下面我们看一下创建协程与启动的源码：

```kotlin
//创建协程
public fun <R, T> (suspend R.() -> T).createCoroutine(receiver: R,completion: Continuation<T>): Continuation<Unit> =
    SafeContinuation(createCoroutineUnintercepted(receiver, completion).intercepted(), COROUTINE_SUSPENDED)

//启动协程
public fun <T> (suspend () -> T).startCoroutine(
    completion: Continuation<T>
) {
    createCoroutineUnintercepted(completion).intercepted().resume(Unit)
}
```

从上面我们可以看出，创建一个协程需要传入一个 ***Continuation***，但同样它也返回了一个 ***Continuation***（为我们协程创建出的本体）。

总结如下：

- ***suspend*** 函数本身执行需要一个 ***Continuation*** 实例在恢复时调用，即此处的参数：***completion***
- 返回值 **Contination<Unit>** 则是创建出来的协程的载体，receiver **suppend** 函数会被传给该实例作为协程的实际执行体。



### 协程上下文(CoroutineContext)

```kotlin
   suspend {  }.startCoroutine(object:Continuation<Unit>{
       override val context: CoroutineContext
           get() = TODO("not implemented") //To change initializer of created properties use File | Settings | File Templates.

       override fun resumeWith(result: Result<Unit>) {
           TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
       }
   })
```

![image-20200108212822271](https://tva1.sinaimg.cn/large/006tNbRwly1gaph5c6x3gj30kx075ab5.jpg)

- 协程执行过程中需要携带数据
- 索引是 CoroutineContext.key
- 元素是 CoroutineContext.Element



### 拦截器

- 拦截器 ***ContinuationInterceptor*** 是 协程上下文(CoroutineContext) 元素
- 可以对协程上下文所在协程的 ***Contiunation*** 进行拦截

![image-20200108213546179](https://tva1.sinaimg.cn/large/006tNbRwly1gaphd1nxhrj30qc0afdhx.jpg)



#### Continuuation执行流程图



  协程挂起恢复要点

- 协程体内的代码都是通过 Continuation.resuumeWith 调调用
- 每调用一次 label 加1，每一个挂起点对应一个 case 分支
- 挂起函数在返回 CORUTINE_SUUSPENDED 时才会挂起



## 协程的启动模式

| 启动模式     | 功能特性                                                     |
| ------------ | ------------------------------------------------------------ |
| DEFAULT      | 立即开始调度协程体，调度前若取消则直接取消                   |
| ATOMIC       | 立即开始调度，且在第一个挂起点不能取消                       |
| LAZY         | 只有在需要(start/join/await)时开始调度                       |
| UNDISPATCHED | 立即在当前线程执行协程体，直到遇到第一个挂起点(后面取决于调度器) |



#### 其他特性

- Channel 热数据流，并发安全的通信机制
- Flow :冷数据流，协程的响应式 API
- Select 可以对多个挂起事件进行等待



## Channel

- 非阻塞的通信基础设置
- 类似于 BlickingQueue+挂起函数