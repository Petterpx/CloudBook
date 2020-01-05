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

  

#### 私有属性的 set，get权限

```kotlin
//
class Person(var age: Int, var name: String) {
     var fitstName: String = ""
        private set(value) {
            field = name
        }
}
```



#### 顶级声明的可见性

- 顶级声明指文件内直接定义的属性，函数，类等
- 顶级声明不支持 **protected**
- 顶级声明被 **private** 修饰表示文件内部可见

 



## 延迟初始化  

- 类属性必须在构造时初始化
- 某些成员只有在类构造之后才会被初始化

1. latteinit 会让编译器忽略变量的初始化，不支持Int等基本类型
2. 开发者必须能够完全确定变量值的生命周期下使用 Iateinit

有如下三种方式：

#### lazy，推荐方式

```kotlin
 private val rvMainActivity by lazy {
        findViewById<RecyclerView>(R.id.rv_main)
}
```



#### lateinit，使用时需注意null处理

```kotlin
  private lateinit var recyclerView:RecyclerView
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        recyclerView=findViewById(R.id.rv_main)
    }
```



#### ？，不推荐，每次都需要?或者！！

```kotlin
  private var recyclerView:RecyclerView?=null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        recyclerView=findViewById(R.id.rv_main)
        recyclerView?.adapter=RecyclerCustomAdapter(arrayListOf("123","234"))
    }
```



## 代理

####  Delegates.observable

> 用于监控属性值发生变更，类似一个观察者，当属性值被修改后往外部抛出一个变更的回调。

```kotlin
//使用state变量代理StateManager，从而属性更改时实现监听
class StateManager{
    var state:Int by Delegates.observable(0){
        property, oldValue, newValue ->
        println("$oldValue->$newValue")
    }
}


fun main() {
    var stateManager = StateManager()
    stateManager.state=4
    stateManager.state=5
}
```



#### 属性代理

#### By

```kotlin

fun main(){
  //属性x将它的访问器逻辑委托给了X对象
    var x:Int by X() 
}   

    
//遵循getValue,setValue，如果是val,只支持getValue
class X{
    operator fun getValue(nothing: Nothing?, property: KProperty<*>): Int {
            return   2
    }

    operator fun setValue(nothing: Nothing?, property: KProperty<*>, i: Int) {


    }
}
```



#### Delegates.notNull()

> 需要在使用前初始化

```kotlin
//返回具有 非Null参数的委托
 var name:String by Delegates.notNull()
```



#### Delegates.vetoble

> 用于在属性改变发生是通知外部变化，在里面可以做一些改变,返回一个状态，如果满足条件返回true,否则false

```kotlin
 var vetoable: Int by Delegates.vetoable(0) { property, oldValue, newValue ->
        println("old-$oldValue---new-$newValue")
        return@vetoable newValue == 123
    }
```



#### Delegates.observable

> 用于在属性发生变化时调用指定的回调函数

```kotlin
   var observable:Int by Delegates.observable(0){
        property, oldValue, newValue -> 
        
    }
```



## 使用属性代理读写 Propertie

```kotlin
class PropertiesDelegate(private val path: String, private val defaultValu: String = "") {
    private lateinit var url: URL

    private val properties: Properties by lazy {
        val prop = Properties()
        url = try {
            javaClass.getResourceAsStream(path).use {
                prop.load(it)
            }
            javaClass.getResource(path)!!
        } catch (e: Exception) {
            try {
                ClassLoader.getSystemClassLoader().getResourceAsStream(path).use {
                    prop.load(it)
                }
                ClassLoader.getSystemClassLoader().getResource(path)!!
            } catch (e: Exception) {
                FileInputStream(path).use {
                    prop.load(it)
                }
                URL("file://${File(path).canonicalPath}")
            }
        }
        prop
    }

    operator fun getValue(thisRef: Any?, kProperty: KProperty<*>): String {
        return properties.getProperty(kProperty.name, defaultValu)
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        properties.setProperty(property.name, value)
        File(url.toURI()).outputStream().use {
            properties.store(it, "petterp")
        }
    }
}

abstract class AbsProperties(path: String) {
    protected val prop = PropertiesDelegate(path)
}

class Config : AbsProperties("Config.properties") {
    var name by prop
}

fun main() {
    val config=Config()
    println(config.name)
    config.name="asdasd"
    println(config.name)
}
```



## 单例 Object

```kotlin
//饿汉式单例
object Singleton{

}
```

```
便于Java调用
@JvmStatic 生成静态方法
@JvmField 生成set,get
```



####   伴生对象

>   Kotlin中没有static变量，所以使用伴生对象代替静态变量。使用前需要带上 相应注解

```kotlin
class Test{

    companion object{
        fun of(){
            
        }
    }
}
```



## 内部类

```kotlin
class Outher{
    //非静态内部类
    inner class Inner{
        
    }
    
    //静态内部类
    class staticInner{
        
    }
}
```



#### 内部Object

```kotlin
class Outher {
    //非静态内部类
    inner class Inner {

    }

    //静态内部类
    class staticInner {
        object OutherObject {
            var x=5
        }
    }
}


fun main() {
    Outher.staticInner.OutherObject.x
}
```



#### 匿名内部类

```kotlin
fun main() {
  	//可继承父类或实现多个接口
  	//类型-交叉类型 ClassPath&Runnable（猜测）
    object : ClassPath(), Runnable {
        override fun run() {
        }

        override fun test() {
        }
    }
}

abstract class ClassPath {
    abstract fun test()
}
```



## 数据类

#### Data  

```kotlin
data class Book(val id:Int,val money:Double){
    
}
```



#### 与JavaBean的区别

|               | **JavaBean**                    | **data class**     |
| ------------- | ------------------------------- | ------------------ |
| **构造方法**  | **默认无参构造**                | **属性作为参数**   |
| **字段**      | **字段私有，Getter/Setter公开** | **属性**           |
| **继承性**    | **可继承也可被继承**            | **不可被继承**     |
| **Component** | **无**                          | **相等性，解构等** |



#### 如何合理使用data class

- 使用基本类型作为属性参数



## 枚举类

```kotlin
enum class State{
    A,B
}

enum class States(val id:Int){
    Idle(0),Busy(1)
}

//枚举实现接口
enum class ClassPath : Runnable {
    IDLE,BUSY;

    override fun run() {
    }
}

//为每一个枚举实现接口方法
enum class EnumOffer : Runnable {
    idle {
        override fun run() {
        }
    },
    busy{
        override fun run() {
        } 
    }
    override fun run() {
    }
   
}
```



#### 为枚举定义扩展

```kotlin
fun State.sout() {
    println("扩展")
}
```



#### 枚举的比较

```kotlin
enum class State {
    A, B
}

fun main() {
    val s = State.B
    if (s>State.A) println("ok")
}
```



#### 枚举的区间

```kotlin
enum class State {
    A, B,C,D,E
}
fun main() {
    val s = State.B
    if (s in State.A..State.C) println("ok")
}
```



## 密封类

-  密封类是一种特殊的抽象类
- 密封类的子类定义在自身相同的文件中
- 密封类的子类个数有限

> 简单来说，密封类相当于一类事物的具体子分类，有明确的类型区别，子类有具体个数。在when表达式中，对密封类有优化效果

```kotlin
sealed class Book(val name: String)

class Android(name: String) : Book(name)
class Java(name: String, var money: Int) : Book(name)

fun main() {
    list(Android("Kotlin"))
    list(Java("Javaxx", 99))
}

//在条件全部满足的情况下，无需else
fun list(book: Book) = when (book) {
    is Android -> println(book.name)
    is Java -> println("${book.name} --  ${book.money}")
}
```



#### 密封类与枚举类的区别

|          | 密封类   | 枚举类   |
| -------- | -------- | -------- |
| 状态实现 | 子类继承 | 类实例化 |
| 状态可数 | 子类可数 | 实例可数 |
| 状态差异 | 类型差异 | 值差异   |



## 内联类 inline class

- 内联类是对某一个类型的包装
- 内联类是类似于 Java 装箱类型的一种类型
- 编译器会尽可能使用被包装的类型进行优化



#### 内联类使用场景

##### 无符号类型

```kotlin
inline class Unit constructor(internal val data: Int) : Comparable<Unit> {
    override fun compareTo(other: Unit): Int {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }

}
```



##### 内联类模拟枚举

```kotlin
inline class State(val ordinal: Int) {
    companion object {
        val Idle = State(0)
        val Busy = State(1)
    }

    fun values() = arrayOf(Idle, Busy)
    val name: String
        get() = values()[ordinal].name
}
```



#### 模拟枚举优化

```kotlin
inline class RouteRypeInline(val valueL:Int)

object RouteTypes{
    val WALK=RouteRypeInline(0)
    val BUS=RouteRypeInline(1)
    val CAR=RouteRypeInline(2)
}
fun setRouterType(routeRypeInline: RouteRypeInline){
    
}


fun main() {
    setRouterType(RouteTypes.BUS)
}
```



#### 内联类的限制

- 主构造器必须有且仅有一个只读属性
- 不能定义有backing-fiield的其他属性
- 被包装类型必须不能是泛型类型
- 不能继承父类也不能被继承
- 内联类不能定义为其他类的内部类 



#### typealias与inline class的区别

- Typelias 相当于起了一个别名
- inline class相当于多了一层包装

|          | Typealias    | Inline class       |
| -------- | ------------ | ------------------ |
| 类型     | 无新类型     | 有包装类型产生     |
| 实例     | 与源类型一致 | 必要时使用包装类型 |
| 使用场景 | 类型更加直观 | 优化包装类型性能   |



## Kotlin的Json序列化

使用Gson,MoShi,K.S

```kotlin
@JsonClass(generateAdapter = true)
@Serializable
data class Person(val name: String, val age: Int = 18)

@ImplicitReflectionSerializer
fun main() {
    var s1 =0
    var s2 =0
    var s3 =0
    for (i in 1..100000){
        s1+= gson()
        s2+= moshi()
        s3+= kotlinks()
    }
    println("gson $s1")
    println("moshi $s2")
    println("kotlinx $s3")
}

fun gson(): Int {
    val currentTimeMillis = System.currentTimeMillis()
    val gson = Gson()
    println(gson.toJson(Person("Petterp")))
    println(gson.fromJson("{\"name\":\"petterp\",\"age\":20}", Person::class.java))

    return (System.currentTimeMillis() - currentTimeMillis).toInt()

}

fun moshi(): Int {
    val currentTimeMillis = System.currentTimeMillis()

    val moshi = Moshi.Builder()
        .build()
    val jsonAdapter = moshi.adapter(Person::class.java)
    println(jsonAdapter.toJson(Person("petterp", 20)))
    println(jsonAdapter.fromJson("{\"name\":\"petterp\",\"age\":20}"))
    return (System.currentTimeMillis() - currentTimeMillis).toInt()

}

@ImplicitReflectionSerializer
fun kotlinks(): Int {
    val currentTimeMillis = System.currentTimeMillis()


    println(Json.stringify(Person("name", 20)))
    println(Json.parse<Person>("{\"name\":\"petterp\",\"age\":20}"))
    println("Kotlinx:" + (System.currentTimeMillis() - currentTimeMillis))

    return (System.currentTimeMillis() - currentTimeMillis).toInt()


}
```

配置模板

#### Gson

```kotlin
implementation 'com.google.code.gson:gson:2.8.6'
```

#### MoShi

```kotlin
apply plugin: 'kotlin-kapt'

implementation("com.squareup.moshi:moshi-kotlin:1.9.2")
kapt 'com.squareup.moshi:moshi-kotlin-codegen:1.8.0'
```

#### kotlinx

```kotlin
apply plugin: 'kotlinx-serialization'
ext.kotlin_version = '1.3.61'

implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
implementation "org.jetbrains.kotlinx:kotlinx-serialization-runtime:0.14.0"
```

```groovy
ext.kotlin_version = '1.3.61'
classpath "org.jetbrains.kotlin:kotlin-serialization:$kotlin_version"
```



#### 性能对比

- 测试设备 mbpro2018 Android Studio3.5
- 测试次数10w次，最终结果

```kotlin
gson 2557
moshi 1386
kotlinx 1511
```



#### 框架对比

|        | Gson      | MoShi      | K.S  |
| ------ | --------- | ---------- | ---- |
| 空类型 | 否        | 反射，注解 | 是   |
| 默认值 | 否        | 反射，注解 | 是   |
| init块 | NoArg插件 | 反射，注解 | 是   |
| Java类 | 是        | 是         | 否   |
| 跨平台 | 否        | 否         | 是   |





## 递归整型列表

```kotlin
sealed class IntList {
    object Nil : IntList() {
        override fun toString(): String {
            return "Nil"
        }
    }

    data class Cons(val head: Int, val tail: IntList) : IntList() {
        override fun toString(): String {
            return "$head,$tail"
        }
    }

    fun joinToString(sep: Char = ','): String {
        return when (this) {
            Nil -> "Nil"
            is Cons -> {
                "$head$sep${tail.joinToString(sep)}"
            }
        }
    }
}

fun IntList.sum(): Int {
    return when (this) {
        IntList.Nil -> 0
        is IntList.Cons -> head + tail.sum()
    }
}

operator fun IntList.component1(): Int? {
    return when (this) {
        IntList.Nil -> null
        is IntList.Cons -> head
    }
}

operator fun IntList.component2(): Int? {
    return when (this) {
        IntList.Nil -> null
        is IntList.Cons -> tail.component1()
    }
}

operator fun IntList.component3(): Int? {
    return when (this) {
        IntList.Nil -> null
        is IntList.Cons -> tail.component2()
    }
}

fun initListOf(vararg ints: Int): IntList {
    return when (ints.size) {
        0 -> IntList.Nil
        else -> {
            IntList.Cons(
                ints[0],
                initListOf(*(ints.slice(1 until ints.size).toIntArray()))
            )
        }
    }
}

fun main() {
    val list = initListOf(0, 1, 2, 3)
    println(list.toString())
    println(list.joinToString('-'))
    println(list.sum())

    //解构
    val (first, second, third) = list
    println(first)
    println(second)
    println(third)

//    val (a, b, c, d, e) = listOf<Int>()
}
```

