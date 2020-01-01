# Kotlin-类型再进阶

## 类的构造器

```kotlin
fun main() {
    Kang::name
    Kang::money //报错
}

//加了var或者val 就相当于成员变量，全局可见，否则就只能在构造器(init块)内可见,类似于局部变量
class Kang(var name:String,money:Int)
```



### init块

init会在最后合成为一个init块，会与构造方法一起执行

```kotlin
class Kang(var name:String,money:Int){
    init {
        this.name=money.toString()
    }

    init {
        println()
    }
    
    init {
        println("我是Petterp")
    }
}
```



### 副构造器

在Java里面，一个类可以有多个构造器，也就意味着我们初始化这个类的方法有很多种，这也就意味着我们有很多生命了的变量无法被使用到。

```kotlin
class Kang(override var name:String, money:Int):Test(name){
    init {
        this.name=money.toString()
    }

    init {
        println()
    }

    init {
        println("我是Petterp")
    }
    fun void(){
    }


    //副构造器，同时调用了主构造器
    constructor(name: String):this(name,123){
        this.name=name
    }
}  

```



如果未定义主构造器，则可以直接调用其他副构造器，和java基本没有什么不同.不过这种写法并不推荐

```kotlin
class Test3{
    constructor(name:String){
        
    }
    
    constructor(money:Int):this("123"){
        
    }
}
```



Kotlin推荐以主构造器+默认参数的形式去写。如果要用在Java中，则加上 **@JvmOverloads**

```kotlin
class Test3 constructor(){
    constructor(name:String){

    }

    @JvmOverloads
    constructor(money:Int):this("123"){

    }
}
```





## 类与成员的可见性

| 可见性类型      | Java                 | Kotlin                           |
| --------------- | -------------------- | :------------------------------- |
| ***public***    | ***公开***           | ***与java相同，默认就是public*** |
| ***internal***  | ***x***              | ***模块内可见***                 |
| ***default***   | ***包内可见，默认*** | ***x***                          |
| ***protected*** | ***包内及子类可见*** | ***类内及子类可见***             |
| ***private***   | ***类内可见***       | ***类或文件内可见***             |

#### internal的小技巧

在Java中可以直接访问到 internal ,因为Java并不认识Kotlin中的 internal。如下，两个模块中

**Kotlin:**

<img src="https://tva1.sinaimg.cn/large/006tNbRwly1gag1elrwunj309w03twej.jpg" alt="image-20191231173335878" style="zoom:110%;" />

**Java：**

![image-20191231173432841](https://tva1.sinaimg.cn/large/006tNbRwly1gag1fkwzc5j30f404j3yr.jpg)



如果我们想避免Java直接访问到我们的代码，可以加入以下小技巧,这样当Java调用时就会因不规范而报错。

![image-20191231173528846](https://tva1.sinaimg.cn/large/006tNbRwly1gag1gk15gdj30ai03pjri.jpg)



![image-20191231173552689](https://tva1.sinaimg.cn/large/006tNbRwly1gag1gyxve8j30dw03bq35.jpg)





