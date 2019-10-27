# Kotlin中内联函数的作用是什么？

#### Android每日一问，小聚成河，大聚成江。

#### 更多请访问GitHub  [Android每日面试题总结](https://github.com/Moosphan/Android-Daily-Interview)。

> 注：以下为我个人理解与大家回答整理，不定时更新最新回答。

---

> 在以前，因为学过一段时间Kotlin(并没有实际开发中用过),很多东西都忘记了，但是kotlin的代码看起来其实和Java没什么区别，感觉都差不多。所以不要认为 Kotlin 很难学。

### 首先，什么是内联函数 inline？

Kotlin的内联函数属于Kotlin的高级特性之一，使用起来也非常简单。

下面，我用实际操作来演示吧；

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(sum(10)+sum(5));
    }

    private static int sum(int a){
        int sum=8;
        sum+=a;
        return sum;
    }
}
```

这是一段java代码，简单的不能再简单了吧，就是重复的相加，别注意逻辑，只是为了演示。

**同样的kotlin代码：**

```kotlin
inline fun sum(a: Int): Int {
    var sum = 8
    sum += a
    return sum
}

fun main() {
    println(sum(10)+ sum(5))
}
```

虽然一眼看上去很简洁，但我们的关注点不在这里，在 inline 关键字上面。为了便于大家学习，我通过查看字节码的方式来转成相应的 java 代码，便于大家更好的理解。

**没加 inline 之前**

![image-20191021220114042](https://tva1.sinaimg.cn/large/006y8mN6ly1g86658a6tbj30l009h3zo.jpg)

**加上 inline 之后**

![image-20191021220149030](https://tva1.sinaimg.cn/large/006y8mN6ly1g8692z19zbj30k20du75f.jpg)

解释就不用多说了吧，kotlin 自动帮我们将方法在编译期就加在了相应的调用处，免除了 java 中的入方法栈与退栈。

> PS：(不要觉得kotlin好难，其实我也是现学现卖，虽然以前也看过一点基础，哈哈)

下面我们再扩展一些知识：



以下源于大家的回答，我并不能明白具体原因，所以需要周末补课。

TODO

#### noinline

> 让原本的内联函数形参函数不是内联的，保留原有数据特征

如果一个内联函数的参数里包含 lambda表达式，也就是函数参数，那么该形参也是 inline 的，举个例子：

inline fun test(inlined: () -> Unit) {...}
这里有个问题需要注意，如果在内联函数的内部，函数参数被其他非内联函数调用，就会报错，如下所示：

```JAVA
noinline
如果一个内联函数的参数里包含 lambda表达式，也就是函数参数，那么该形参也是 inline 的，举个例子：
inline fun test(inlined: () -> Unit) {...}
这里有个问题需要注意，如果在内联函数的内部，函数参数被其他非内联函数调用，就会报错，如下所示：
```

要解决这个问题，必须为内联函数的参数加上 noinline 修饰，表示禁止内联，保留原有函数的特性，所以 test() 方法正确的写法应该是：

```kotlin
inline fun test(noinline inlined: () -> Unit) {
otherNoinlineMethod(inlined)
}
```



#### crossinline

> 非局部返回标记
> 为了不让lamba表达式直接返回内联函数，所做的标记
> 相关知识点：我们都知道,kotlin中,如果一个函数中,存在一个lambda表达式，在该lambda中不支持直接通过return退出该函数的,只能通过return@XXXinterface这种方式

首先来理解一个概念：非局部返回。我们来举个栗子：

```kotlin
fun test() {
innerFun {
//return 这样写会报错，非局部返回，直接退出 test() 函数。
return@innerFun //局部返回，退出 innerFun() 函数，这里必须明确退出哪个函数，写成 return@test 则会退出 test() 函数
}

//以下代码依旧会执行
println("test...")
}
```

```kotlin
fun innerFun(a: () -> Unit) {
a()
}
```



非局部返回我的理解就是返回到顶层函数，如上面代码中所示，默认情况下是不能直接 return 的，但是内联函数确是可以的。所以改成下面这个样子：

```kotlin
fun test() {
innerFun {
return //非局部返回，直接退出 test() 函数。
}
```

```kotlin
inline fun innerFun(a: () -> Unit) {
a()
}
```



也就是说内联函数的函数参数在调用时，可以非局部返回，如上所示。那么 crossinline 修饰的 lambda 参数，可以禁止内联函数调用时非局部返回。

```kotlin
fun test() {
innerFun {
return //这里这样会报错，只能 return@innerFun
}

//以下代码不会执行
println("test...")
}

inline fun innerFun(crossinline a: () -> Unit) {
a()
}
```



#### 具体化的类型参数 reified

> java中，不能直接使用泛型的类型
> kotlin可以直接使用泛型的类型

```kotlin
inline fun startActivity() {
startActivity(Intent(this, T::class.java))
}
使用时直接传入泛型即可，代码简洁明了：
```

