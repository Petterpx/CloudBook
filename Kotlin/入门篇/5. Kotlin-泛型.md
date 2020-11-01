# Kotlin-泛型

对一个Android开发者来说，不会使用泛型，简直是一种莫大的耻辱，而大多数人对于泛型的使用也仅仅停留在基本的<T>层次。下面，当我们开始学习Kotlin泛型的时候，建议认真去回顾一下Java的泛型，泛型擦除等相关，再来学习Kotlin泛型



## Java泛型

- ？extends   传递的类型必须继承指定的类，并且不支持后期修改

-   ?  super  将父类类型赋值给子类泛型声明



### 泛型的概念

- 泛型是一种类型层面的抽象
- 泛型通过泛型参数实现构造更加通用的类型的能力
- 泛型可以让符合继承关系的类型批量实现某些能力

实际使用

```kotlin
fun <T> maxOf(a: T, b: T): T {

}

class Demo<T> {

}
```



## 泛型约束

> 让我们的类是某一个类的子类

```kotlin
fun <T:Comparable<T>> maxOf(a:T,b:T):T{
    return if (a>b) a else b
}
```



```kotlin
interface InTest {
    fun sout()
}

open class TestF

class TestFItem : TestF(), InTest {
    override fun sout() {
        println(213)
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }
}

//泛型约束，T必须实现InTest接口,并且继承TestF
//如果加了where 请将条件语句放到where中统一管理
fun <T> InTestFun() where T : InTest, T : TestF {
}

fun main(){
  InTestFun<TestFItem>()
}
```

#### 对于泛型的一些理解demo

```kotlin
fun main() {
    open1(::open1Fun)
    open2(Test("petterp", 1)::test)
}

//泛型约束
//（）-> 此时传递的就必须是一个函数，并且无参无返回值
fun <T> open1(t: T) where T : () -> Unit = t()

//多参数
//() -> R 此时传递的为一个无参返回值为R类型的函数
//当多参数时，此时可能需要将对应的泛型个数拆分为相应的类
fun <T, R> open2(t: T): R where T : () -> R = t()

fun open1Fun() = println("单参数泛型约束")


class Test<T, R>(var r: R, var t: T) {
    fun test(): R {
        println("返回R类型")
        return r
    }
}

```



## 泛型型变

- 泛型实参的继承关系对泛型类型的继承关系的影响
  1. 协变：继承关系一致   ? extends xxx
  2. 逆变：继承关系相反   ? super xxx
  3. 不变：没有继承关系



### 协变点

函数返回值类型为泛型参数

```kotlin
//out 协变，T 一定是Book子类
class BookStore<out T : Book> {
    fun getBook(): T {
    }
}

fun convariant() {
    val eduBookStore: BookStore<EduBook> = BookStore()
    val bookStore: BookStore<Book> = eduBookStore

    //拿到父类
    val book: Book = bookStore.getBook()
    //拿到子类
    val eduBook: EduBook = eduBookStore.getBook()
}


fun main() {

}
```

### 协变小结

- 子类 xx 兼容父类 Base
- 生产者 Producer<Derived> 兼容 Producer<Base>
- 存在 **协变点** 的类的泛型参数必须声明为 协变或不变
- 当泛型类作为泛型参数类实例的 **生产者** 时用 **协变**



### 逆变

> 简单来说，逆变相当于一种限定，限制类型为固定的

```kotlin
open class Waste

class DryWaste : Waste()

class Dustbin<in T : Waste> {
    fun put(t:T){

    }
}

fun main() {
    val dustbin: Dustbin<Waste> = Dustbin()
    val drwWaste: Dustbin<DryWaste> = dustbin

    val waste=Waste()
    val dryWaste=DryWaste()

    dustbin.put(waste)
    dustbin.put(dryWaste)

    //报错，只能存放指定DryWaste类型
    drwWaste.put(dustbin)
}
```

-  子类 兼容 父类 Base
- 消费者 Consumber<Base> 兼容 Consumer<子类>
- 存在逆变点的类的泛型参数必须声明为 逆变或不变



## UnsafeVariance

###  在开发中，违反型变约束的安全前提

违反型变约束

- 即声明 为 协变的 类出现逆变点 或相反
- 声明为 不变 的类接收 逆变 或 协变 的类型参数

安全前提

- 泛型参数协变，逆变点不能引起修改，即始终 只读不写
- 泛型参数协变，协变点不得外部获取，即始终 只写不读



示例

```kotlin
class Dustbin<in T> {
    val list = mutableListOf<@UnsafeVariance T>()
    fun put(t:T){

    }
}
```

Kotlin中的 List不是可变的，因此如果类型是一个协变点，则会报错，需要加上 @UnsafeVariance注解

![image-20200105182055659](https://tva1.sinaimg.cn/large/006tNbRwly1galuvfa3zqj30hk03n74m.jpg)



### 简单理解

![image-20200105182156070](https://tva1.sinaimg.cn/large/006tNbRwly1galuwfg14sj32840s049t.jpg)





## * 星投影

-    可用在变量类型声明的位置
- 可用以描述一个未知的类型
- 所替换的类型在：
  - 协变点返回泛型参数上限类型
  - 逆变点接收泛型参数下限类型

```kotlin
class QueryMap<out K : CharSequence, out V : Any> {

    fun getKey(): K = TODO()

    fun getValue(): V = TODO()
}

class Function<in P1, in P2> {
    fun invoke(p1: P1, p2: P2) = Unit
}


fun main() {
  //根据是否是协变和逆变来决定上限和下限
  
    //out取上限
    val queryMap: QueryMap<*, *> = QueryMap<String, Int>()
    queryMap.getKey()
    queryMap.getValue()

    //in 取下限
    val f: Function<*, *> = Function<Number, Any>()
    f.invoke()
}

```



### 使用范围

- 不能直接或间接应用在函数或函数上
  - QueryMap<String,*>()
  - maxOf<*>(1,3)
- 适用于作为类型描述的场景
  - **val queryMap:QueryMap<*,*>**
  - **if (f is Funcation<*,*>){...}**
  - **HashMap<String,List<*>>() **



## 泛型的实现类型与内联优化

### 泛型实现原理

- 伪泛型：编译时擦除类型，运行时无实际类型生成

  例如：Java,Kotlin

- 真泛型：编译时生成真实类型，运行时也存在该类

  例如：C#,C++

### reified 内联特化

只能用于inline处



## 模拟Self Type

```kotlin
interface SelfType<out Self> {
    val self: Self
        get() = this as Self
}

open class NotificationBuilder<Self> : SelfType<Self> {

    private var title: String = ""
    private var content: String = ""
    fun title(title: String): Self {
        this.title = title
        return self
    }

    open fun build(): Notification = Notification(title)
}

class Notification(private var title: String) {
    fun test() {
        println(title)
    }
}


fun main() {
    Test1().title("123").build().test()
}

class Test1 : NotificationBuilder<Test1>() {

}
```

> 从而可以实现在使用时可以拿到子类，便于进行一些额外操作



##  使用泛型注入Model

![image-20200105222110821](https://tva1.sinaimg.cn/large/006tNbRwly1gam1tddupuj30r006575g.jpg)

报错已解决，原因是getValue方法中的thisRef参数，也就是当前属性对象需要null处理。加上?即可.

新问题todo:为什么在类中无需添加?，而在类中定义一个方法，然后继续使用 局部属性代理 依然报错，不太明白？

> 一些基础 
>
> operator fun <T : AbsModel> getValue(thisRef: Any?, property: KProperty<*>): T {
>         return Models.run {
>             //capitalize首字母大写
>             property.name.capitalize().get()
>         }
>     }
>
> > 对于只读属性(也就是说val属性), 它的委托必须提供一个名为getValue()的函数。
> > thisRef: Any? 必须是该属性当前类或者基类(Any是任何类的基类)。
> > property: KProperty<*>这个参数的类型必须是 KProperty<*> , 或者是它的基类。
> > getValue()返回值类型必须与属性类型相同(或者是它的子类型)。
> > value:<类型>这个参数的类型必须与属性类型相同, 或者是它的基类
> > 方法名不能改必须是getValue、setValue，并且必须用operator关键字修饰
>
> 参考https://www.jianshu.com/p/30541df7829a

已解决。**事实上属性代理对于getValue或者setValue中的 thisRef类型没有明确要求，如果属性代理是局部变量，则它是不会绑到函数所在的类上，所以局部变量的情况时，thisRef 一定是null.**

```kotlin
fun initModels() {
    DatabaseModel()
    NetworkModel()
    SpModel()
}

object Models {
    private val modelMap = ConcurrentHashMap<String, AbsModel>()

    fun AbsModel.register(name: String = this.javaClass.simpleName) {
        modelMap[name] = this
    }

    fun <T : AbsModel> String.get(): T {
        return modelMap[this] as T
    }

}


object ModelDelegate {
    operator fun <T : AbsModel> getValue(thisRef: Any?, property: KProperty<*>): T {
        return Models.run {
            //capitalize首字母大写
            property.name.capitalize().get()
        }
    }

}


/*
属性代理
* */
class MainViewModel {
    val databaseModel: DatabaseModel by ModelDelegate
    val networkModel: NetworkModel by ModelDelegate
    val spModel: SpModel by ModelDelegate
    val spModel2: SpModel by ModelDelegate
}


abstract class AbsModel {
    init {
        Models.run {
            register()
        }
    }
}

class DatabaseModel : AbsModel() {
    fun query(sql: String): Int = 0
}

class NetworkModel : AbsModel() {
    fun get(url: String) = "{co}"
}

class SpModel : AbsModel() {
    init {
        Models.run {
            register("SpModel2")
        }
    }

    fun hello()= println("123")
}



fun main() {
    //注入Model
    initModels()
    val mainViewModel = MainViewModel()
    mainViewModel.databaseModel.query("123").let(::println)
    mainViewModel.networkModel.get("www.baidu.com").let(::println)
    mainViewModel.spModel.hello()
    mainViewModel.spModel2.hello()
}
```

