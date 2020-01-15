# Kotlin-协程是什么？

Hi你好，新同学。很高兴，你终于追寻这个问题了，也许你正感到迷茫，各路大神对协程的理解不一，有人说它是线程框架，有人说它比线程更轻，希望我这篇博文可以帮你从另一个角度简单理解协程。



请相信一句话，任何解释从第二个人口中说出时，可能已经存在了变化。而官网是我们接触任何技术最必要的门槛。所以请打开[Kotlin中文网](https://www.kotlincn.net/docs/reference/coroutines-overview.html)。很多人说kotlin官网教程很不详细，其实不然，kotlin中文网教程很详细。回到正题：



## 什么是协程？

![image-20200115165921441](https://tva1.sinaimg.cn/large/006tNbRwly1gaxcpmvvuxj30n006n3zv.jpg)

- 异步编程
- 体验
- 语言级
- 理念

注意上面几个关键点和一些实际使用，不难明白

Kotlin协程是基于Kotlin语法从而延伸的一个异步编程框架，它并没有带来多少性能上的体验，它能实现的，你用线程池同样也可以实现，但对于使用角度的来说，协程努力打造一个 ***"同步方式，异步编程的"*** 思想，作为开发者来说，我们可以更懒了，切换线程，withContext即可，协程带来了开发上的舒适，但这种舒适是基于 Kotlin 的语法，并不是别的。所以我希望大家关注协程时，多从语言角度去理解。

##### 那么，协程是什么？

> **协程就是一个基于Kotlin语法的异步框架，它可以使开发者以同步的方式，写成异步的代码，而无需关注多余操作。就这么简单**

---

## 协程怎么用？

#### 启动一个协程

```kotlin
suspend fun main() {
    GlobalScope.launch {
        println("启动--${System.currentTimeMillis()}---${Thread.currentThread().name}")
        delay(100)
        println("挂起100ms--${System.currentTimeMillis()}---${Thread.currentThread().name}")
    }
    delay(1000)
}
```

```
启动--1579085375166---DefaultDispatcher-worker-1
挂起100ms--1579085375275---DefaultDispatcher-worker-1
```



#### 并发使用

```kotlin
fun main() {
    runBlocking(Dispatchers.IO) {
        println("启动--${System.currentTimeMillis()}")
        val as1 = async {
            delay(100)
            println("并发1--${System.currentTimeMillis()}---${Thread.currentThread().name}")
            "1"
        }
        val as2 = async {
            delay(100)
            println("并发2--${System.currentTimeMillis()}---${Thread.currentThread().name}")
            "2"
        }
        println("as1=${as1.await()},as2=${as2.await()}")
    }
}
```

```kotlin
启动--1579085205199
并发1--1579085205313---DefaultDispatcher-worker-3
并发2--1579085205313---DefaultDispatcher-worker-2
as1=1,as2=2
```



观察上面demo的运行结果，是不是很舒服，看起来同步的方式内部却是在异步操作。那上面注释中 **挂起** 是什么意思呢？



## 什么是挂起？

观察上面的打印日志，我们不难发现，在调用 delay 函数时，线程并没有停下，相对来说，只是我们的协程代码块被挂起，等待恢复。只有前面的挂起函数执行结束，我们的协程代码块才能继续执行。借用一幅图来说明如下：

![img](https://tva1.sinaimg.cn/large/006tNbRwly1gaxggh8lzag30oo0dw4ko.gif)

**所以所谓的挂起其实是代码层次的一个处理，从而使得我们可以以同步形式去写异步的代码。**



## 非阻塞程序？

所谓的非阻塞，其实就是切换了线程，观察打印日志变化，我们可以发现，当我们直接 GlobalScope.launch 启动一个协程时，此时运行的线程为默认的线程，所以协程被称为非阻塞的实现方式。

#### 

### suspend关键字的作用

**先看下面的图**

![image-20200115191719199](https://tva1.sinaimg.cn/large/006tNbRwly1gaxgp581iuj30ci06rmxm.jpg)

当使用suspend修饰后的函数我们称其为挂起函数，**那么挂起函数有什么作用？为什么test 的suspend 标志是黑的？**

##### 继续看截图

![image-20200115192201091](https://tva1.sinaimg.cn/large/006tNbRwly1gaxgu161jcj30eo05ht92.jpg)

#### **挂起函数为什么只能在挂起函数中使用呢？？**

我们再继续往下看：**看一下java字节码**

![image-20200115193946330](https://tva1.sinaimg.cn/large/006tNbRwly1gaxhci80umj30lg078jsk.jpg)

这个 **Continuation**是什么呢？按照字面意思，意思为延续。那我们该怎么理解呢？

一般来说 **Continuation** 代表的是<剩余的计算的概念>，也就是 接下来要执行的代码。官方的解释叫做 **挂起点**

比如我有一段代码：

```kotlin
println("123".length)
```

最先执行的是"123".length，然后执行 println打印长度，此时执行 **"123".length** 就可以当做一个挂起点，也就是代码从这里停止，等待计算出结果，然而此时内部线程却没有停止，当计算完的时候，也就是挂起结束，此时接着执行我们的打印语句。

切换到我们的suspend中，它代表的就是一个***标志*** 的作用，有suspend修饰的代表的函数叫做挂起函数，当编译器碰到这个标志的函数，就知道它是一个可能会耗时的操作。



#### **挂起函数为什么只能在挂起函数中使用呢？**

![image-20200115200403816](https://tva1.sinaimg.cn/large/006tNbRwly1gaxi1rw13kj30l306f3ze.jpg)

因为普通函数参数并没有带 Continuation啊，相当于没有挂起点，编译器无法判断，所以此时会报错。



#### **为什么test 的suspend 标志是黑的？**

> 编译器知道它是挂起函数，但是test内部没有挂起函数啊，所以此时编译器提示。
>
> **那这个函数就一点作用没有？**
>
> ***有作用，就是限制这个函数只能挂起函数调用。***



## 在Android中使用

### 倒计时功能

```kotlin
class Main3Activity : AppCompatActivity() {
    private val mainScope = CoroutineScope(Dispatchers.Default)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main3)
        progressBar.max = 100
        mainScope.launch {
            (1..100).forEach {
                delay(100)
                withContext(Dispatchers.Main) {
                    progressBar.secondaryProgress = it
                }
            }
        }

    }
  
  onDestory(){
    //记得关闭
  }
  
}
```

![Jietu20200115-203101-HD](https://tva1.sinaimg.cn/large/006tNbRwly1gaxiunqobqg30go0b1u10.gif)



### 结合 ViewModel 使用

```kotlin
class MainViewModel : ViewModel() {
    fun test() {
        viewModelScope.launch {

        }
    }
}
```

在ViewModel取消时，协程将自动关闭



### 结合 Lifecycle 使用

> 导入以下依赖
>
> ```
> implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0-rc03"
> ```

于是刚才的倒计时代码改为如下：

```kotlin
class Main3Activity : AppCompatActivity() {
  			....
        lifecycleScope.launch {
           ...
        }
    }
}
```

同样，当fragment或者Activity关闭时，协程同样将自动关闭。

**查看源码，会发现，viewModel中的 viewModelScope 和 Lifecycle lifecycleScope,实现方式如出一辙：**

![image-20200115204534210](https://tva1.sinaimg.cn/large/006tNbRwly1gaxj92o6hcj32po0qmnpd.jpg)





***本篇，我们没有过多的从源码上去追寻，协程到底是什么，尽量从语法，使用者的角度入手，带着大家从侧面去理解协程，希望大家都能有自己的理解。***

**如果希望深入研究，推荐以下博客：**

[Beenyhuo](https://www.bennyhuo.com/)

