# Kotlin-函数再探索

## 高阶函数

什么是高阶函数？

> - 参数包含函数类型
> - 返回了一个函数类型

如下：

```kotlin
fun main() {
    //调用，传入一个函数
    test2(1, test1(9))
}

//返回类型为函数的叫做高阶函数和
fun test1(arg: Int): (Int) -> Int =
    {
        println("返回类型为函数")
        arg + 9999
    }


//参数为函数的叫做高阶函数
fun test2(a: Int, b: (Int) -> Int) {
    println("参数为函数")
    println(a + b(a))
}
```

```
参数类型为函数类型
返回类型为函数
10009
```



### 常见的高阶函数

#### forEach

```kotlin
listOf(1,2).forEach(::println)
listOf("petterp","kotlin").forEach { println(it) }
```

![image-20191230164458970](https://tva1.sinaimg.cn/large/006tNbRwly1gaeudpej8gj30lo05jmxn.jpg)



#### map

```kotlin
 mapOf("Kotlin核心技术" to 99.9,"Androidxxx" to 88).map {
        println("${it.key} --- ${it.value}")
    }
```

![image-20191230165026220](https://tva1.sinaimg.cn/large/006tNbRwly1gaeujdwadfj30ud05zjsa.jpg)





## 内联函数

使用准则：内联函数与高阶函数配合使用效果更佳

- 函数本身被内联到调用处
- 函数的函数参数被内联到调用处

高阶函数的return

```kotlin
   listOf("petterp","kotlin","123").forEach {
        if (it=="petterp") return@forEach   //类似continue
        if (it=="kotlin") return            //直接跳出
        println(it)
    }
```



noinline  禁止函数参数被内联



**内联函数的限制**

- public/protected 的内联方法只能访问对应类的 public 成员
- 内联函数的内联函数参数不能被存储(赋值给变量)
- 内联函数的内联函数参数只能传递给其他内联函数参数





## 一些常用的高阶函数

### let

使用场景:

- null类型的判定
- 用于某个变量特定作用域的处理

返回类型以函数最后一行为决定

```kotlin
     val person:Person?=null
    person?.let{
        it.toString()
    }
```



### run

直接可以在run里面都某个变量进行更改，同时具有判null效果

```kotlin
 person?.run {
        x="name"
        y=123
    }
```



### also

与let最大的不同是，also返回的依然是原来的对象

```
//用于修改数据
person?.also {
    it.x="asda"
}
```



### apply

返回类型是原对象，但可以直接在函数内对数据进行更改

```kotlin
var block: Person.() -> Unit = {
        x = "123"
        y = 123
    }
```



### with

在with函数内，可以免除类名，直接调用这个类的相关方法

```kotlin
    class Book{
        fun saveName() {}
        fun getMoney() {}
    }

    with(Book()){
        saveName()
        getMoney()
    }
```

[感谢这个博主：](https://blog.csdn.net/u013064109/article/details/78786646)



### use

```kotlin
   //use自动关闭资源
    File("Kotlin.iml").inputStream().reader().buffered()
        .use {
            println(it.readText())
        }
```





## 集合与变换

### forEach

| **filter**  | **保留满足条件的元素**                                       |
| :---------: | ------------------------------------------------------------ |
|   **Map**   | **集合中的所有元素一一映射到其他元素构成新集合**             |
| **flatMap** | **集合中的所有元素一一映射到新集合，并合并这些集合得到新集合** |



### filter

相当于一个过滤的作用，满足条件的保留

```kotlin
val list= listOf("123","asdads")
println(list.filter { it.length>4 }.toString())
```



### map

映射集合中的所有元素,并转换为一个新的集合

```kotlin
list.map {
    it+"asd"
}.forEach(::println)
```



### flatMap

用于将多个map铺开，然后合并为一个新的map

```kotlin
val list= listOf(1..10,2..10,3..10)
 list.flatMap {
     it.map {
         it+99
     }
 }.forEach(::println)
```



### asSequence（懒序列）

```kotlin
  list.map {
        println(1)
        it+"asd"
    }

    //懒序列 asSequence
    //懒序列是指，如果最终不调用forEach，那么最开始的map也不会执行
    list.asSequence().map {
        println(1)
        it+"asd"
    }.forEach(::println)
```



### 聚合操作

| **sum**    | **所有元素求和**                                         |
| ---------- | :------------------------------------------------------- |
| **Reduce** | **将元素依次按规矩聚合，结果与元素类型一致**             |
| **Fold**   | **给定初始化值，将元素按规矩聚合，结果与初始值类型一致** |

### fold

```kotlin
    println(listOf(1,2,3).fold(5){
            acc, i ->acc+i
    })
```

![image-20191230231208752](https://tva1.sinaimg.cn/large/006tNbRwly1gaf5kjeo5mj30s803ygm0.jpg)



### Reduce

```kotlin
println(listOf(1, 2, 3).reduce { acc, i ->
        acc + i
    })
```

![image-20191230231732952](https://tva1.sinaimg.cn/large/006tNbRwly1gaf5q653vyj30tr0680to.jpg)



### sum

```kotlin
println(listOf(1, 2, 3).sum())
```

![image-20191230231839186](https://tva1.sinaimg.cn/large/006tNbRwly1gaf5rb8jhej30d804xaab.jpg)



### zip

```kotlin
//返回一个列表，这个列表由两个集合中相同索引元素建立的元素数
    listOf(1,2,3).zip(listOf(5,3,1,5)).forEach(::println)
```

```java
(1, 5)
(2, 3)
(3, 1)
```

![image-20191230233028135](https://tva1.sinaimg.cn/large/006tNbRwly1gaf63lruj0j30pg03cdg3.jpg)



//更多集合相关操作，参考https://www.jianshu.com/p/d627bf3d5d72



## SAM

 