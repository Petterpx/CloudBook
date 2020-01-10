# Kotlin-协程使用手册

## 什么是协程？

挂起，恢复，这里所说的挂起恢复指的是在线程内



挂起，恢复听起来很生硬，那该怎么理解呢？

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