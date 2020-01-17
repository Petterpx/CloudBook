# Kotlin-协程使用手册

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

对流进行累加,数据累加

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

对流进行累加,数据累加。不同于 reduce 的是，fold 可以赋初值

```kotlin
runBlocking {
        (1..3).asFlow()
            .fold(10) {acc, i -> acc + i }.let(::println)
    }

```

```
16
```





### 流是连续的

在kotlin中，流是按照顺序执行的。从上游到下游的每个过渡操作符都会处理每个发射出的值然后再交给末端操作符。

简单理解就是，从上到下顺序执行，只有满足上游条件才会执行下面操作符。

```kotlin
 (1..5).asFlow()
        .filter {
            println("filter=$it")
            it % 2 == 0
        }
        .map {
            println("map=$it")
            "map-$it"
        }
        .collect {
            println(
                "collect-$it"
            )
        }
```

```kotlin
filter=1
filter=2
map=2
collect-map-2
filter=3
filter=4
map=4
collect-map-4
filter=5
```



### Flow中的错误示例

在协程中，通常使用 withContext 切换上下文 (**简单理解切换线程，不过也并不准确，因为协程的上下文包含很多数据，如value等，我们通常只是用来切换线程**) ,但是 flow{} 构建器中的代码必须遵循上下文保存属性(**即不允许更改上下文**)，并且不允许从其他上下文中发射数据 (**不允许从其他launch{}发射**)。

> ***但可以通过 flowOn更改***

#### 错误案例1: **更改上下文**

```kotlin
fun main() {
    runBlocking {
        flow {
            withContext(Dispatchers.Default) {
                for (i in 1..3) {
                    delay(100)
                    emit(i)
                }
            }

        }.collect().let(::println)
    }
}
```

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@1f88ae32, BlockingEventLoop@32db9536],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@77a8c47e, DefaultDispatcher].
		Please refer to 'flow' documentation or use 'flowOn' instead
...
```



#### 错误案例2：**从别的上下文发射数据**

```kotlin
fun main() {
    runBlocking {
        flow {
            launch {
                emit(123)
                emit(123)
                emit(123)
            }
            delay(100)

        }.collect {

            println(it)
        }
    }
}
```

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Emission from another coroutine is detected.
		Child of "coroutine#2":StandaloneCoroutine{Active}@379619aa, expected child of "coroutine#1":BlockingCoroutine{Active}@cac736f.
		FlowCollector is not thread-safe and concurrent emissions are prohibited.
		To mitigate this restriction please use 'channelFlow' builder instead of 'flow'
	at 
	...
```



### flowOn

用于更改流发射的上下文。

```kotlin
fun main() {
    runBlocking {
        logThread()
        flow {
            for (i in 1..3) {
                logThread()
                emit(i)
                delay(10)
            }
        }.flowOn(Dispatchers.IO).collect {
            println(it)
        }
    }
}
```

```kotlin
当前所在线程----main @coroutine#1
当前所在线程----DefaultDispatcher-worker-1 @coroutine#2
1
当前所在线程----DefaultDispatcher-worker-1 @coroutine#2
2
当前所在线程----DefaultDispatcher-worker-1 @coroutine#2
3
```

> 这里我们收集在主线程中，发射数据在IO线程。也意味着我们收集与发射此时处于两个协程之中。



### Buffer

流的发射与收集通常是按顺序执行，通过上面我们发现，将流 的不同部分运行在不同的协程中将对于时间有大幅度减少。但现在如果我们不使用 flowOn,此时发射一个流(emit)和收集流(collect)的耗时将累加起来。

> **比如发射一个流需要100ms,收集需要200ms,则发送3个流并收集总需要至少900ms+**

```kotlin
fun main() {
    runBlocking {
        val start=System.currentTimeMillis()
        flow {
            for (i in 1..3) {
                emit(i)
                delay(100)
            }
        }.collect {
            delay(300)
            println(it)
        }
        println("花费时间-${System.currentTimeMillis()-start}ms")
    }
}
```

```kotlin
1
2
3
花费时间-1230ms
```

我们可以在流上使用 ***buffer*** 操作符来并发运行 数据发射及收集，而不是按顺序执行

更改代码如下：

```kotlin
fun main() {
    runBlocking {
        val start = System.currentTimeMillis()
        flow {
            for (i in 1..3) {
                logThread()
                emit(i)
                delay(100)
            }
        }.buffer().collect {
            logThread()
            delay(300)
            println(it)
        }
        println("花费时间-${System.currentTimeMillis() - start}ms")
    }
}
```

```kotlin
当前所在线程----main @coroutine#2
当前所在线程----main @coroutine#1
当前所在线程----main @coroutine#2
当前所在线程----main @coroutine#2
1
当前所在线程----main @coroutine#1
2
当前所在线程----main @coroutine#1
3
花费时间-961ms
```

***我们发现，实则buffer也是内部切换了线程，也就是说buffer和 flowOn使用了相同的缓存机制，只不过 buffer 没有显改变上下文。***



### 合并

#### conflate

用于跳过中间的值，只处理最新的值。

```kotlin
fun main() {
    measureTimeMillis {
        runBlocking {
            (1..5).asFlow()
                .conflate()
                .buffer()
                .collect {
                    delay(100)
                    println(it)
                }
        }
    }.let{
    println("花费时间-${it}ms")
    }
}
```

```kotlin
1
5
花费时间-325ms
```



### 处理最新值

#### collectLatest & conf

取消缓慢的收集器，并在每次发射新值的时候重新启动它。

```kotlin
fun main() {
    measureTimeMillis {
        runBlocking {
            (1..5).asFlow()
                .collectLatest {
                    println(it)
                    delay(100)
                    println("重新发射-$it")
                }
        }
    }.let {
        println("花费时间-${it}ms")
    }
}
```

```
1
2
3
4
5
重新发射-5
花费时间-267ms
```



### 组合流

#### zip

组合两个流中的相关值

```kotlin
fun main() {
    measureTimeMillis {
        val strs= flowOf("one","two","three")
        runBlocking {
            (1..5).asFlow()
                .zip(strs){
                    a,b ->

                    "$a -> $b"
                }.collect { println(it) }
        }
    }.let {
        println("花费时间-${it}ms")
    }
}
```

```
1 -> one
2 -> two
3 -> three
花费时间-120ms
```



#### Combine

```kotlin
suspend fun main() {
    val nums = (1..3).asFlow().onEach { delay(300) } // 发射数字 1..3，间隔 300 毫秒
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // 每 400 毫秒发射一次字符串
    val startTime = System.currentTimeMillis() // 记录开始的时间
    nums.combine(strs) { a, b -> "$a -> $b" } // 使用“zip”组合单个字符串
        .collect { value -> // 收集并打印
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}
```

```
1 -> one at 472 ms from start
2 -> one at 682 ms from start
2 -> two at 874 ms from start
3 -> two at 987 ms from start
3 -> three at 1280 ms from start
```



## 通道Channel

- 非阻塞的通信基础设施 
- 类似于 BlockingQueue+suspend

提供了一种在 Flow 中传递数据的方法

### Channel的分类

| 分类       | 描述                                             |
| ---------- | ------------------------------------------------ |
| RENDEZVOUS | 不见不散，send调用后挂起直到receive 到达         |
| UNLIMITED  | 无限容量，send调用后直接返回                     |
| CONFLATED  | 保留最新，reveive只能获取最近一次的 send 的值    |
| BUFFERED   | 默认容量，可通过该程序参数设置默认大小，默认为64 |
| FIXED      | 固定容量，通过参数执行缓存大小                   |



### Channel基础

```kotlin
suspend fun main() {
   runBlocking {
       val channel = Channel<Int>()
       launch {
           // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
           for (x in 1..5) channel.send(x * x)
       }
// 这里我们打印了 5 次被接收的整数：
       repeat(5) { println(channel.receive()) }
       println("Done!")
   }
}
```

```
1
4
9
16
Done!
```



### channel的关闭与迭代

```kotlin
suspend fun main() {
    runBlocking {
        val channel = Channel<Int>()
        launch {
            // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
            for (x in 1..5) {
                channel.send(x * x)
            }
            channel.close()
            println("通道是否关闭+${channel.isClosedForSend}")
        }
        for (i in channel) {
            println(i)
        }
        println("是否接收了所有的数据+${channel.isClosedForReceive}")
    }
}
```

```
1
4
9
16
25
```





### consumeEach

用以代替for循环

```kotlin
suspend fun main() {
   runBlocking {
       val channel = Channel<Int>()
       launch {
           // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
           for (x in 1..5) {
               channel.send(x * x)
           }
           channel.close()
       }
       channel.consumeEach(::println)
   }
}
```

### 

### Channel的协程Builder

- Produce 启动一个生产者协程，返回ReceiveChannel
- actor :  启动一个消费者协程，返回 SendChannel （暂时废弃）
- 以上Builder启动的协程结束后悔自动关闭对应的 Channel

#### Produce

```kotlin
suspend fun main() {
    val receiveChannel = GlobalScope.produce(capacity = Channel.UNLIMITED) {
        for (i in 0..3) {
            send(i)
        }
    }

      GlobalScope.launch {
        receiveChannel.consumeEach(::println)
    }.join()

}
```

```
0
1
2
3
```



#### actor

```kotlin
suspend fun main() {
    val sendChannel = GlobalScope.actor<Int>(capacity = Channel.UNLIMITED) {
        channel.consumeEach(::println)
    }

    GlobalScope.launch {
        for (i in 1..3){
            sendChannel.send(i)
        }
    }.join()

}
```



### BroadcastChannel

- Channel 的元素只能被一个消费者消费
- BroadcastChannel 的元素可以分发给所有的订阅者
- BroadcastChannel 不支持RENDEZVOUS

```kotlin
suspend fun main() {
    val broadcastChannel = GlobalScope.broadcast {
        for (i in 1..3)
            send(i)
    }

    List(3) {
        GlobalScope.launch {
            val receiveChannel = broadcastChannel.openSubscription()
            println("-----$it")
            receiveChannel.consumeEach(::println)

        }
    }.joinAll()
}
```

```
-----2
-----0
-----1
1
1
1
2
2
3
3
2
3
```



## Select(实验性)

- 是一个IO多路复用的概念
- koltin中用于挂起函数的多路复用

 Select表达式可以同时等待多个挂起函数，并选择第一个可用的



#### 在Channel使用

```kotlin
suspend fun main() {
    runBlocking {
        val fizz = fizz()
        val buzz = buzz()
        repeat(7) {
            selectFizzBuzz(fizz, buzz)
        }
        coroutineContext.cancelChildren()
    }
}

suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> {
        // <Unit> 意味着该 select 表达式不返回任何结果
        fizz.onReceive { value ->
            // 这是第一个 select 子句
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->
            // 这是第二个 select 子句
            println("buzz -> '$value'")
        }
    }
}

@ExperimentalCoroutinesApi
fun CoroutineScope.fizz() = produce {
    while (true) {
        delay(300)
        send("Fizz")
    }
}

@ExperimentalCoroutinesApi
fun CoroutineScope.buzz() = produce {
    while (true) {
        delay(500)
        send("Buzz")
    }
}
```

```
fizz -> 'Fizz'
buzz -> 'Buzz'
fizz -> 'Fizz'
fizz -> 'Fizz'
buzz -> 'Buzz'
fizz -> 'Fizz'
buzz -> 'Buzz'
```

> 使用receive 挂起函数，我们可以从两个通道中接收其中一个数据，但是select允许我们使用其 onReceive 子句同时从两者接收。
>
> ***注意：onReceiver 在已经该关闭的通道执行会发生失败并抛出异常，我们可以使用onReceiveOrNull 子句在关闭通道时执行特定操作***