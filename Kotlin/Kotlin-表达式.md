# Kotlin-表达式

## 常量和变量

```kotlin
var a=1
a=2

//编译器常量
//只能定义全局范围，只能修饰基本类型。必须立即赋值
//const 类似于 static
const val b=33

//运行时常量
val h=5
```



## 分支表达式

```kotlin

fun main() {
    val a = 3
    //if表达式
    val b = if (a == 4) 4 else 5

    //when表达式
    when (a) {
        0 -> 5
        1 -> 10
        else -> println()
    }

    //when分支表达式
    when{
        a>3 -> println("")
        else -> println()
    }

    //表达式try-catch
    val number=try {
        println(".....")
    }catch (e:Exception){
        0
    }

}
```



## 中缀表达式与运算符

```kotlin
fun main{ 
//常见的扩展方法
    println("123".rotate1(5))
    //使用中缀表达式
    //a to(b) ，只有一个参数
    println("123" rotate2 5)
}

fun String.rotate1(count: Int): String = count.toString()
infix fun String.rotate2(count: Int): String = count.toString()

```



## 重写enquals和hashcode

```kotlin
fun main() {
    val map=HashSet<Prestion>(5)
    (0..5).forEach {
        map+=Prestion(1,2)
    }
//    var prestion =Prestion(2, 2)
//    map+=prestion
//    prestion.a=10
//    val prestion1=Prestion(prestion.a+1,prestion.b)
//    map-=prestion1
    println(map.size)
}


class Prestion(var a: Int, var b: Int) {

    override fun equals(other: Any?): Boolean {
        val outher = (other as? Prestion) ?: return false
        return outher.a == a && outher.b == b
    }

    override fun hashCode(): Int {
        return 1+a*b+7
    }
}
```





## 简单的四则运算demo

```kotlin
fun main() {

    val value = "123"
    println(value + "546")
    println(value - "1")
    println(value * 10)
    println(value / "12")
}

//-
//replaceFirst 替换字符串，按照给定的条件替换，默认只替换第一次
operator fun String.minus(right: Any?) = this.replaceFirst(right.toString(), "")

//*
//joinToString 根据给定的条件创建一个新的字符串，默认使用 "," 隔开
operator fun String.times(right: Int) = (1..right).joinToString("") { this }

//+
operator fun String.plus(right: Any?) = this + right

// /
//windowed 按照指定条件滑动，每次滑动指定位置(这里为1)，当满足条件时返回true,最终返回类型时一个list,再通过count统计为true的item
operator fun String.div(right: String) = this.windowed(right.length, 1) { it == right }.count { it }

```

**结果**

```java
123546
23
123123123123123123123123123123
1
```

