# Kotlin-内置类型

## 基本类型

![image-20191228150348866](https://tva1.sinaimg.cn/large/006tNbRwly1gacg7vo022j31gx0u0wzy.jpg)

```
var b: String = "asdasd"
val c: Int = 15
```



字符串比较，

==,===

== 比较内容是否相等

=== 比较对象是否相等



## 数组

![image-20191228155051782](https://tva1.sinaimg.cn/large/006tNbRwly1gachksd2yyj31f10u0k9d.jpg)

```kotlin
//it代表当前数组下标
var intArray = IntArray(3) { it +3}

//获取长度
println(intArray.size)
//打印数组
println(intArray.contentToString())
//打印数组
val dd= arrayOf(1,2,3)
println("${dd[1]},${dd[2]}")
//按数组下标打印
val ss= intArrayOf(1,2,3,4);
    for (i in ss.indices){
        println(ss[i])
}

//数组遍历
for (d in dd){
        print(d)
}
//函数式写法
 dd.forEach { d->
        println(d)
 }
//如果使用默认it,则可以省略 d->
dd.forEach { println(it)}


/* 判断是否在一个数组内 */
   if (5 in dd){
        println("在数组dd内")
    }

    if (5 !in dd){
        println("不在数组dd内")
    }


```





## 区间

```kotlin
//表示创建了一个1-10的闭区间
val intRange=1..10
val charRange='a'..'z'
val longRange=1L..1000L

//创建开区间
val intRangeExclusive=1 until  10 //[1,10)

//倒序区间
    val intRangeReverse=10 downTo  1 //[10,9,...,1]

//区间添加步长
val intRange1=1..10 step 3 // [1,4,7,9,10]
 //打印区间
println(intRange1.joinToString())

```



## 集合框架

![ image-20191228170007121](https://tva1.sinaimg.cn/large/006tNbRwly1gacjkuaixgj31cy0u01kx.jpg) 

```kotlin
//不可变List,不可添加或删除
val intList: List<Int> = listOf(1,2,3)
//可变List
val intList2: MutableList<Int> = mutableListOf(1, 2, 3, 4)

//不可变map  
val map:Map<String,Any> = mapOf("name" to "petterp","ts" to 123)
val mutilMap:Map<String,Any> = mutableMapOf("name" to "petterp","Ts" to 123)


//直接创建Java中的List
val intList = ArrayList<Int>()
val stringList = ArrayList<String>()

val intList = ArrayList<Int>()
    //List的写入 等价于 add
    for (i in 0..10){
        intList+=i
    }
    //删除某个元素,等价于 remove
    intList.remove(100)
    //修改某个元素的值 类似于 set(0,101)
    intList[0]=101
    

val maps = HashMap<String, String>(10)
    //添加某个元素
    maps+=("key1" to "test")
    println(maps.toString())
    //修改某个元素
    maps["key1"]="petterp"
    println(maps.toString())

```

Any 相当于Object



如果我们查看 Kotlin中的 ArrayList,会发现

![image-20191228180401420](https://tva1.sinaimg.cn/large/006tNbRwly1gaclfd1jjrj30tt05eq4h.jpg)

**其中 Typealias** 相当与为类型起新名称



我们在上面Map中用到了 ”“ to ""

Var pair=""

```kotlin
 val map = HashMap<String, String>()

    val pair = "key" to "petterp"
    val pair2 = Pair("Hello", "Petterp")
    //获取键
    val key = pair.first
    //获取值
    val second = pair.second
    //puts如map
    map += pair
    map += pair2
    println("$key , $second")
```



### 什么是解构？

用于将一个对象解构成多个变量

[使用案例如下](https://www.kotlincn.net/docs/reference/multi-declarations.html)

```java
fun main() {

    val map = HashMap<String, Int>()
    val kotlinBook = "Kotlin核心技术" to 88
    val AndroidBook = Pair("Android艺术探索", 66)
    println("${kotlinBook.first} , ${kotlinBook.second}")
    map += kotlinBook
    map += AndroidBook

    println("解构开始\n\n")
    /*解构声明*/
    //参考https://www.kotlincn.net/docs/reference/multi-declarations.html
    //将一个对象解构为多个变量
    println("解构对象")
    val (x, y) = Book("Kotlin核心技术", 88)
    println(x)
    println(y)


    println("\n 遍历map")
    //使用解构遍历map,可能是最好的方式
    for ((name, value) in map) {
        println("$name , $value")
    }

    println("\n 忽略某些解构")
    //如果你不想解构某些变量，则通过 _ 标志
    for ((name, _) in map)
        println(name)

    println("\n lambda中使用解构")
    //使用解构返回一个新的map,key不变，返回的只是value的改变
    val maps = map.mapValues { (name, value) -> "5" }
    println(map.toString())
    println(maps.toString())

    println("\n 函数中使用解构")
    //从函数中返回两个变量
    val (name, money) = funcation()
    println("$name , $money")
    //从函数中返回两个变量
}

data class Book(var name: String, var money: Int)


fun funcation(): no1.Book {
    println("经历了一波操作")
    return no1.Book("Android艺术探索", 99)
}

```

```html
Kotlin核心技术 , 88
解构开始


解构对象
Kotlin核心技术
88

 遍历map
Kotlin核心技术 , 88
Android艺术探索 , 66

 忽略某些解构
Kotlin核心技术
Android艺术探索

 lambda中使用解构
{Kotlin核心技术=88, Android艺术探索=66}
{Kotlin核心技术=5, Android艺术探索=5}

 函数中使用解构
经历了一波操作
Android艺术探索 , 99

```



## Kotlin函数

#### :: 函数引用

```kotlin
fun main() {
    //函数引用的几种方法
    val x = Test::pp
    val x1: (Test, String) -> String = Test::pp
    val x2: (Test, String) -> String = x
    val x3: Function2<Test, String, String> = Test::pp
}


class Test {
    fun pp(name: String): String {
        return name
    }
   
}
```



#### 函数作为参数使用

```kotlin
fun main(){
  val x1: (Test, String) -> String = Test::pp
println(Test().pp2(x1, 123))
}

class Test(){
  
  fun pp(name: String): String {
        return name
    }
  
  fun pp2(name: (Test, String) -> String, value: Int): String {
        return "${name(Test(), "132")} , $value"
    }
}
```



#### 默认参数

```kotlin
fun main(){
    //默认参数
    default("Android")
    //如果你的默认参数为第一个，此时就需要声明显示类型
    default(name = "Android")
}

fun default(name: String, money: Int = 9) {

}
```



#### 变长参数

```kotlin
fun main(){
	 //变长参数
    channels("Asd", "Asd")
}
fun channels(vararg args: String) {
    println(args.joinToString())
}
```



#### 多返回值参数

```kotlin
fun main(){
var (a, b, c) = test1()
    println("$a , $b ,$c")
}

fun test1(): Triple<Int, String, String> {
    return Triple(123, "asd", "asdfa");
}
```

