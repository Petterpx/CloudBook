# Kotlin学习，基础学习

## 类和接口

#### 定义一个接口

```kotlin
//定义一个接口
interface SimpleInter {
    fun test()
    val number: Int
}
```



#### 定义一个类

```kotlin
//open表示允许继承，在Kotlin中，类和方法之间默认不允许继承和重写(不包括抽象类)
open class SimpleClass{
		open fun put()
}
```



#### 类之间的继承及实现一个接口

```kotlin
//实现接口中的参数
class Test1(override val number: Int) :SimpleClass(),SimpleInter{
    
    //接口方法
    override fun test() {
    
    }
    
    //重写父类方法
    override fun openTest() {
        super.openTest()
    }

}
```



#### 自定义set,get

```kotlin
//在kotlin中，默认会为参数添加set,get方法，如果需要自定义，按照以下方式写即可
class ClassTest1(override val number: Int) : SimpleInter {
    override fun test() {

    }

    var money = number
        get() {
            return field
        }
        set(value) {
            field = value
        }

}
```



#### 类的实例化及属性的调用

```kotlin
val number=Test1::number

//属性调用
val classTest=ClassTest1(123)
val money=classTest::money
money.set(456)
```





## 扩展方法

```kotlin
class Book{

}

//定义Book类的扩展方法
fun Book.noBook(){
	
}

//也可以定义扩展属性
// Backing Field
//扩展成员变量无法存储状态，因为不存在field
var Book.money: Int
    get() {
        return this.saveInt
    }
    set(value) {
        this.saveInt = value
    }

//接口可以定义变量
interface Guy {
    var moneyLeft: Double
        get() {
            return 0.0
        }
        set(value) {

        }
}




```





## 空类型安全

```kotlin
class Book{

}

fun main() {
    val book=Book()
    if (book is Book){
        //安全的类型转换
        println((book as? Book)?.javaClass?.name)
    }
}
```



