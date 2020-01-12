# Kotlin-协程使用手册

HI，新同学，很高兴你终于看到了这篇博文，在学习之前，我希望你有一定的**基础**，至少使用过。接下来，将从实例开始，带你对协程有一个自己的认知。

***为什么是自己的认知？***

我相信大多数人在学习 协程的时候，都在追寻一个问题，**协程是什么？协程有什么用？**

然后网上各种解释，协程就是一个线程框架，协程节省效率，协程比线程更好使用，协程如何如何，但哪些标题式的解释，我们真的能明白吗，我们为什么不先从语法上入手？然后再去探究它的实现，如果你都不了解它，何谈实现原理？

网上对于协程的理解多数人认为它是一个轻量级的线程，官方也是这样解释的。但是作为开发者，协程并不只是一个简单的线程框架，它更多的是借助于 kt 这门语言，改变我们常规的编码方式(比如java编码习惯)。

举个简单的例子如下：

在 java 中，我们可能不会显式去关注null指针，泛型等问题，但在 kt 中，这些问题，需要开发者强制去处理，对于我们java 中常写的回调来说，回调是再简单不过的事，回调嵌套也是常见的事。但在 kt 中，这种方式有更好的解决方案，那就是 协程。它可以帮你以同步的方式，写出异步的操作，而核心的原则就是 ***挂起，恢复***。这两个词，现在可能听起来很迷，但当你逐步使用协程时，我相信你会有自己的理解。不会纠结于协程到底是什么。

更希望大家从kt 的这门语言上去看协程究竟是什么？它到底能给我们带来哪些畅快体验，从而真正理解 协程



首先，借助于一张图，加深大家对于 挂起，恢复的理解

![img](https://user-gold-cdn.xitu.io/2019/6/20/16b72f0821633f92?imageslim)



### 启动一个协程

```kotlin

fun main() {

    GlobalScope.launch {
        println(123)
    }
    Thread.sleep(10)
}

```



###   阻塞方式等待协程执行完再执行后续

```kotlin

    GlobalScope.launch {
        delay(1000)
        println(123)

    }.join()
    println(1231)
```



### delay

> 特殊的挂起函数，它并不会造成函数阻塞，但是会挂起协程



### 协程作用域构建器

### runBlocking

> 会阻塞当前线程，直到协程结束。 一般用于测试

```kotlin
 runBlocking {
                    launch(Dispatchers.IO) {
                        Log.e("demo", Thread.currentThread().name)
                        delay(10000)
                        println("!23")
                    }
                }
```



### coroutineScope

> 只是挂起，会释放底层线程用于其他用途，并不会阻塞线程。所以我们称它为挂起函数

```kotlin
coroutineScope {
                    launch(Dispatchers.IO) {
                        Log.e("demo", Thread.currentThread().name)
                        delay(10000)
                        println("!23")
                    }
                }
```





### 结构化并发

> 虽然协程使用起来很简单，当我们使用 GlobalScope.launch 时，我们会创建一个顶级协程，但是这样使用也不是我们所推荐的方式，特别是如果我们忘记了对新启动协程的引用，它还是会继续运行。所以在实际应用中，我们更推荐 ： **在执行操作所在指定作用域内启动协程，而非随意使用**





## 协程的取消与超时

### cancelAndJoin

> 取消一个协程并等待结束

```kotlin
   runBlocking {
        val startTime = System.currentTimeMillis()
        val job = launch(Dispatchers.Default) {
            var nextPrintTime = startTime
            var i = 0
            while (i < 5) { // 一个执行计算的循环，只是为了占用 CPU
                // 每秒打印消息两次
                if (System.currentTimeMillis() >= nextPrintTime) {
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500L
                }
            }
        }
        delay(1300L) // 等待一段时间
        println("main: I'm tired of waiting!")
        job.cancelAndJoin() // 取消一个作业并且等待它结束
        println("main: Now I can quit.")
    }
```



### withTimeout

> 超时后抛出异常

```
//超时抛出异常
withTimeout(1300L) {
    delay(1400)
}
```



### withTimeoutOrNull

> 超时后抛出null指针

```
//超时抛出异常
        withTimeoutOrNull(1300L){
            delay(1400)
        }
```



### 在finally中释放资源

```kotlin
coroutineScope {
    val a = launch {
        try {
            repeat(1000) { i ->
                println("日常打印")
                delay(500)
            }
        } finally {
            println("回收资源")
        }
    }
    delay(1000)	//延迟一段时间
    println("延迟结束")
    a.cancelAndJoin()   //取消一个作业并等待它结束
}
```



### 在finally中重新挂起协程

> 在我们实际应用中，可能需要在finally重新挂起一个被取消的协程，所以可以将相应的代码包装在 
>
> **withContext(NoCancellable)**中

```kotlin
coroutineScope {
        val a = launch {
            try {
                repeat(1000) { i ->
                    println("日常打印")
                    delay(500)
                }
            } finally {
                withContext(NonCancellable){
                    println("挂起一个被取消的协程")
                    delay(1000)
                    println("挂起取消")
                }
            }
        }
        delay(1000)
        println("延迟结束")
        a.cancelAndJoin()   //取消一个作业并等待它结束
    }
```



### 超时抛出异常

> 设置超时时间,超过预期时间，抛出异常。可以用于倒计时等

```kotlin
coroutineScope {
    try {
        withTimeout(1000){
            println("超过2000ms就失败")
            delay(2000)
        }
    }catch (e:TimeoutCancellationException){
        println(e.message)
        println("好的好的，我知道了")
    }
}
```

```
超过2000ms就失败
Timed out waiting for 1000 ms
好的好的，我知道了
```



### 超时抛出null指针

> 有些情况，你可能并不想直接抛出异常，则可以让其抛出null指针

```kotlin
   coroutineScope {
        val time = withTimeoutOrNull(1000) {
            println("超过2000ms就失败")
            delay(2000)
        }
        println(time) //null
    }
```

```
超过2000ms就失败
null
```



## 组合挂起函数



### 默认顺序调用挂起函数

**measureTimeMillis** 

```kotlin
suspend fun main() {
    val time = measureTimeMillis {
        playGame()
        playPP()
    }
    println("经过了$time ms")
}

suspend fun playGame() {
    delay(1000)
    println("打豆豆1秒")
}

suspend fun playPP() {
    delay(1000)
    println("打屁屁1秒")
}
```

```koltin
打豆豆1秒
打屁屁1秒
经过了2031 ms
```



### async

**并发执行**

> 在上面的例子中，我们按顺序执行，但我们实际开发中，更多的是希望并行执行，借助于 ***async*** 我们就可以实现。

#### 注意

在概念上，[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 就类似于 [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html)。它启动了一个单独的协程，这是一个轻量级的线程并与其它所有的协程一起并发的工作。不同之处在于 `launch` 返回一个 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) 并且不附带任何结果值，而 `async` 返回一个 [Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html) —— 一个轻量级的非阻塞 future， 这代表了一个将会在稍后提供结果的 promise。你可以使用 `.await()` 在一个延期的值上得到它的最终结果， 但是 `Deferred` 也是一个 `Job`，所以如果需要的话，你可以取消它。

```kotlin
suspend fun main() {
    measureTimeMillis {
        coroutineScope {
            async { playGame() }
            async { playPP() }
        }
    }.let {
        println("所费时间+ $it")
    }
}

suspend fun playGame() {
    delay(1000)
    println("打豆豆1秒")
}

suspend fun playPP() {
    delay(1000)
    println("打屁屁1秒")
}
```

```
打豆豆1秒
打屁屁1秒
所费时间+ 1084
```



### 惰性启动的 async

> 我们可以通过更改 **async** 的属性来实现惰性模式，在这个模式下，只有通过 **await** 或者 async的返回值 **job.start**,才会启动
>
> ***注意：如果直接调用await,那么结果将会是顺序执行***

```kotlin
suspend fun main() {

    measureTimeMillis {
        coroutineScope {
            val game = async(start = LAZY) { playGame() }
            val pp = async(start = LAZY) { playPP() }
            game.start()
            pp.start()
            println("启动完成")
        }
    }.let(::println)
    }

suspend fun playGame() {
    delay(1000)
    println("打豆豆1秒")
}

suspend fun playPP() {
    delay(1000)
    println("打屁屁1秒")
}
```



### async 风格的函数

> 我们可以定义异步风格的函数来异步调用 playGame 和 playPP，并使用 async 协程建造器并带有一个显式的 GlobalScope引用

```kotlin
suspend fun main() {
  measureTimeMillis {
        val somethingPlayGame = somethingPlayGame()
        val somethingPlayPP = somethingPlayPP()
        runBlocking {
            somethingPlayGame.await()
            somethingPlayPP.await()
        }
    }.let(::println)
}
fun somethingPlayGame() = GlobalScope.async {
    playGame()
}

fun somethingPlayPP() = GlobalScope.async {
    playPP()
}

suspend fun playGame() {
    delay(1000)
    println("打豆豆1秒")
}

suspend fun playPP() {
    delay(1000)
    println("打屁屁1秒")
}
```

```kotlin
打屁屁1秒
打豆豆1秒
1085
```

***注意：这样的写法我们并不推荐。如果  val somethingPlayGame = somethingPlayGame() 和 somethingPlayGame.await() 有逻辑错误，程序将抛出异常，但是 somethingPlayPP 依然在后台执行，但是因为前者异常，所有协程都将被关闭，所以 somethingPlayPP 操作也会终止***

**案例如下**：

```
...

suspend fun playGame() {
    delay(500)
    throw ArithmeticException("error")
    println("打豆豆1秒")
}

...
```

```kotlin
Exception in thread "main" java.lang.ArithmeticException: error
	at com.xiecheng_demo.java.TestKt.playGame(Test.kt:46)
	....
```



### 使用async正确的结构化并发

> 按照协程的层次结构传递

```kotlin
suspend fun main() {
    runBlocking {
        try {
            test()
        } catch (e: ArithmeticException) {
            println("main-${e.message}")
        }
    }
}

suspend fun test() = coroutineScope {
    async { playGame() }
    async { playPP() }
}


suspend fun playGame() {
    try {
        delay(1000)
        println("打豆豆1秒")
    } finally {
        println("我在打豆豆，万一异常了。。。")
    }
}

suspend fun playPP() {
    delay(500)
    throw ArithmeticException("抛出异常")
}
```

```kotlin
我在打豆豆，万一异常了。。。
main-抛出异常
```

***注意：如果其中一个子协程失败，则第一个 playGame 和等待中的父协程都会被取消***



## 协程上下文和调度器

协程总是运行在以 **coroutineContext** 为代表的上下文中，协程上下文是各种不同元素的集合,事实上， **coroutineContext** 就是一个存储协程信息的context

![image-20200111154205812](https://tva1.sinaimg.cn/large/006tNbRwly1gaso00xm4vj30r006ijsf.jpg)



### 调度器

**coroutineContext** 包含了**dispatchers**,我们可以借助其限制协程的工作线程。

分别有如下几种：

- Dispatchers.Default	 协程默认线程
- Dispatchers.IO    io线程
- Dispatchers.Main  主线程
- Dispatchers.Unconfined 无限制，将直接运行在当前线程



### 子协程

当一个协程被其他协程在 CoroutineScope 启动时，它将通过 **CoroutineScope.CoroutineContext** 来承袭上下文，并且这个新协程将成为父协程的子作业。当一个父协程被取消时，同时意味着所有的子协程也会取消。

然而，如果此时用 GlobalScope.launch启动子协程，则它与父协程的作用域将无关并且独立运行。

```kotlin
    val a = GlobalScope.launch {
        GlobalScope.launch {
            println("使用GlobalScope.launch启动")
            delay(1000)
            println("GlobalScope.launch-延迟结束")
        }

        launch {
            println("使用 launch 启动")
            delay(1000)
            println("launch-延迟结束")
        }
    }
    delay(500)
    println("取消父launch")
    a.cancel()
    delay(1000)
```

```
使用GlobalScope.launch启动
使用 launch 启动
取消父launch
GlobalScope.launch-延迟结束
```



### join

使用 join 等待所有子协程执行完任务。

```kotlin
suspend fun main() {
    val a = GlobalScope.launch {
        GlobalScope.launch {
            println("使用GlobalScope.launch启动")
            delay(1000)
            println("GlobalScope.launch-延迟结束")
        }

        launch {
            println("使用 launch 启动")
            delay(1000)
            println("launch-延迟结束")
        }
    }
   a.join()
}
```

在上面main函数中，GlobalScope.launch启动的协程将立即独立执行，如果不使用join,则main可能瞬间执行完成，从而无法看不到效果。使用join方法从而使得 main 所在的协程暂停，直到 GlobalScope.launch 执行完成。



### 指定协程名

使用 ***+*** 操作符来指定

```kotlin
   GlobalScope.launch(Dispatchers.Default+CoroutineName("test")){
        println(Thread.currentThread().name)
    }.join()
```

这里使用了 jvm参数  ***-Dkotlinx.coroutines.debug***

**如何配置jvm参数**:Android Studio,Intellij同理

![image-20200111173719310](https://tva1.sinaimg.cn/large/006tNbRwly1gasrbuu1wrj30xr0bvdiq.jpg)



### 协程作用域

在我们了解了上面的概念之后，我们开始将前面学到的结合在一起。定义一个全局的 协程。

```kotlin
class Main4Activity : AppCompatActivity() {
    private val mainScope = CoroutineScope(Dispatchers.Default)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main4)
      
        btn_start.setOnClickListener {
            mainScope.launch {
                //一波操作后
                launch(Dispatchers.Main) {
                    toast("one")
                }
            }
            Log.e("demo","123")
            mainScope.launch (Dispatchers.Main) {
                delay(3000)
                    toastCenter("two")
            }
        }

        btn_cancel.setOnClickListener {
            onDestrxx()
        }

    }

    fun onDestrxx() {
        mainScope.cancel()
    }
}
```

我们在Activity中声明了一个 CoroutineScope 对象，然后创建了一些作用域，这样当我们Activity destory的时候，就可以全部销毁。

> 这里为了节省代码，仿 **onDestory** 的作用

效果，点击btn1之后，再点击btn2,将只会弹出一个toast,第二个toast将不会弹出



### 线程局部数据

将一些局部数据传递到协程之间通过 ThreadLoacl即可完成。

```kotlin
suspend fun main() {
    val threadLocal = ThreadLocal<String>()
    threadLocal.set("main")
    GlobalScope.launch(Dispatchers.Default+threadLocal.asContextElement(value = "123")) {
        println("thread-${Thread.currentThread().name},value=${threadLocal.get()}")
        println("thread-${Thread.currentThread().name},value=${threadLocal.get()}")
    }.join()
    println("thread-${Thread.currentThread().name},value=${threadLocal.get()}")
    delay(1000)
    println("thread-${Thread.currentThread().name},value=${threadLocal.get()}")
}
```

```
thread-DefaultDispatcher-worker-1 @coroutine#1,value=123
thread-DefaultDispatcher-worker-1 @coroutine#1,value=123
thread-main,value=main
thread-kotlinx.coroutines.DefaultExecutor,value=null
```

> 你可能会疑问**，为什么delay 之后，threadloadl.get为null**?
>
> 请注意main函数前面加了一个 suspend,而main函数内部就相当于协程体，当我们直接调用 GlobalScope.launch 时，它直接独立运行，此时内部的 **coroutineContext** 为我们手动传递的。
>
> 而当我们调用了 delay之后，直接挂起协程，此时我们的main函数中的 **coroutineContext** 即为默认值null,于是get为null



---



## 异步流

***挂起函数可以异步的返回单个值，而如何返回多个计算好的值，这正是 Flow(流)的使用之处***



### 使用 list 表示多个值

```kotlin
fun foo(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    foo().forEach { value -> println(value) } 
}
```

```
1
2
3
```

> **我们可以看到，相应的值是瞬间一起返回的，如果我们需要他们单个返回呢？**



### 使用 Sequence 表示多个值

> 使用 Sequence 可以做到同步的返回数据，但是其同时阻塞了线程

```kotlin
fun main() {
    foo().forEach(::println)
}

fun foo():Sequence<Int> = sequence {
    println(System.currentTimeMillis())
    for (i in 1..3){
        Thread.sleep(300)
      	//yield  产生一个值，并挂起等待下一次调用
        yield(i)
    }
    println(System.currentTimeMillis())

}
```

```kotlin
1578822422344
1  //->间隔300ms
2		//->间隔300ms
3		//->间隔300ms
1578822423255
```



### 挂起函数

使用 suspend 标志的即为挂起函数。对于编译器来说，suspend 只是起到一个标志作用。

> 在我们上面的代码中，suspend 我们经常见。



### Flow

使用list返回结果，意味着我们会一次返回所有值，而使用Sequence虽然可以做到同步返回，但如果有耗时操作，又会阻塞我们的线程。

flow 正是解决上面存在的弊端的存在。

```
fun main() {
    runBlocking {
   			logThread()
        launch {
            logThread()
            for (i in 1..3) {
                println("lauch块+$i")
                delay(100)
            }
        }
        logThread()
        foo().collect { value ->
            println(value)
        }
    }
}

fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun logThread() = println("当前所在线程----${Thread.currentThread().name}")
```

```kotlin
当前所在线程----main @coroutine#1
当前所在线程----main @coroutine#1
当前所在线程----main @coroutine#2  //lauch{} 
lauch块+1
1
lauch块+2
2
lauch块+3
3
```

> **flow**{} 中的代码可以挂起
>
> 使用 emit() 函数发射值
>
> 使用 collect 函数 收集 值。(可以认为是启动)



### 取消Flow

取消一个 Flow ,其实就是取消协程，我们无法直接取消Flow,但可以通过取消Flow 所在的协程达到目的。

> 观察上面的demo，我们如果给 **foo()** 方法中打印所在线程，就会发现，它所在的线程与 runBlocking  为同一个，即 foo() 使用的是 runBlocking 上下文。

我们改动代码如下：

```kotlin
fun main(){
	...
	withTimeoutOrNull(200){
            foo().collect { value ->
                println(value)
            }
        }
  ...
}
...
```

```kotlin
当前所在线程----main @coroutine#1
当前所在线程----main @coroutine#2
lauch块+1
1
lauch块+2
lauch块+3
```

> 为什么 lauch依然运行呢？
>
> 我们在前面已经说过了，launch{}是独立运行一个协程，与父协程无关，所以此时launch{}不受取消影响



### Flow构建器

#### flowOf 

用于定义一个发射固定值集的流

```kotlin
flowOf("123","123").collect{
    value ->
    println(value)
}
```

#### asFlow

用于将各种集合与序列转为Flow

```kotlin
(1..3).asFlow().collect{value-> println(value)}
```



### 过渡性流操作符

#### map

使用map实现数据转换

```kotlin
runBlocking {
        (1..3).asFlow()
            .map {
                delay(1000)
                "f-$it"
            }
            .collect { value -> println(value) }
    }
```



### 转换操作符

#### transform

使用transform ，我们可以在执行异步请求之前发射一个字符串并跟踪这个响应

```kotlin
    runBlocking {
        (1..3).asFlow()
            .transform {
                request ->
                emit("test-$request")
                //耗时操作
                delay(500)
                emit(request)
            }
            .collect { value -> println(value) }
    }
```

```
test-1
1
test-2
2
test-3
3
```



### 限长操作符

> 在 流 触及相应限制的时候会将它的执行取消。协程中的取消操作总是通过抛出异常来执行，这样所有的资源管理函数(try{},finally{}块 会在取消的情况下正常运行

#### take

获取指定个数的发射个数，到达上限将停止发射

```kotlin
  runBlocking {
        (1..3).asFlow()
            .take(1)
            .collect { value -> println(value) }
    }
```

```kotlin
1  //结果只有一个
```



### 末端流操作符

#### toList

```kotlin
//toList
    runBlocking {
        (1..3).asFlow()
            .toList().let(::println)
    }
```

```
[1, 2, 3]
```



#### toSet

```kotlin
//toSet
    runBlocking {
        (1..3).asFlow()
            .toSet().let(::println)
    }
```

```
[1, 2, 3]
```



#### first

将流规约为单个值。即只发送第一个数据

```
 runBlocking {
        (1..3).asFlow()
            .first().let(::println)
    }
```

```
1
```



#### single

将流规约为单个值。即只发送第一个数据，不同的是，如果发送数据大于1个，将抛出 ***IllegalStateException***

```kotlin
 //single
    runBlocking {
        flowOf(1,2).single().let(::println)
      	//flowOf(1).single().let(::println)
    }
```

```kotlin
1 
Exception in thread "main" java.lang.IllegalStateException: Expected only one element
```



#### reduce

对流进行累加。数据累加

```kotlin
runBlocking {
        (1..3).asFlow()
            .reduce { a, b -> a + b }.let(::println)
    }
```

```
6
```



#### fold

对流进行累加。数据累加。不同于 reduce 的是，fold 可以赋初值

```kotlin
runBlocking {
        (1..3).asFlow()
            .fold(10) {acc, i -> acc + i }.let(::println)
    }

```

```
16
```





