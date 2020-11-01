#  Koltin-反射

## 反射的基本

- 反射是允许程序在运行时访问程序结构的一类特性
- 程序结构包括：类，接口，方法，属性等语法特性

![image-20200106105557716](https://tva1.sinaimg.cn/large/006tNbRwly1gamnmpnmlfj30ho0bp0um.jpg)

反射需要单独依赖相应的库。



### 反射常见用途

- 列出类型的所有属性，方法，内部类等等
- 调用给定名称及签名的方法或访问指定名称的属性
- 通过签名信息获取泛型实参的具体类型
- 访问运行时注解即其信息完成注入或者配置操作



### 反射常用的数据结构

| 数据结构  | 概念及使用说明                                               |
| --------- | ------------------------------------------------------------ |
| KType     | 描述为擦除的类型或泛型参数等，例如Map<String,Int>;    可通过typeOf或者以下类型获取对应的父类，属性，函数参数与等 |
| KClass    | 描述对象的实际类型，不包含泛型参数，例如MAP；可通过对象，类型名直接获得 |
| KProperty | 描述属性，可通过属性引用，属性所在类的KClass获取             |
| KFunction | 描述函数，可通过函数引用，函数所在类的 KClass 获取           |



### Kotlin中使用反射

- Java反射
  - 优点：无需引入额外依赖，首次使用速度相对较快
  - 缺点：无法访问Kotlin 语法特性，需对Kotlin 生成的字节码足够了解
- Kotlin反射
  - 优点：支持访问Kotlin 几乎所有特性，API设计更好
  - 缺点：引入Kotlin 反射库(2.5m,编译后 400kb)，首次调用慢

#### Demo

```kotlin
@ExperimentalStdlibApi
fun main() {
    println("\n反射获取A类对象")
    val a = A::class

    println("\n获取构造函数")
    a.constructors.forEach(::println).let(::println)

    a.isAbstract.let {
        println("是否是抽象类+$it \n")
    }

    println("\n获取该内内部声明的所有类，包括静态类与内部类")
    a.nestedClasses.forEach(::println)

    println("\n类的完全限定的点名分隔符，如果类是本地的或者是匿名对象文字，则为*或`null`。 *")
    a.qualifiedName.let(::println)

    println("\n此类的父类")
    a.supertypes.forEach(::println)
  
  	//获取此类所有的超类
  	// allSupertypes

    println("\n此类的泛型列表。该列表不包括外部类的泛型参数")
    a.typeParameters.forEach(::println)

    println("\n此类的所有非扩展函数")
    a.declaredMemberProperties.forEach(::println)

    println("\n此类的所有函数")
    a.declaredFunctions.forEach(::println)
  

}

open class AAA


open class AA<R> : AAA()


class A<T>(var x: Int) : AA<T>() {
    fun String.heelo() = run {}

    class A1

    fun Funtest() = run { }
}
```

```java
反射获取A类对象

获取构造函数
fun <init>(kotlin.Int): com.no9.A<T>
kotlin.Unit
是否是抽象类+false 

获取该内内部声明的所有类，包括静态类与内部类
class com.no9.A$A1

类的完全限定的点名分隔符，如果类是本地的或者是匿名对象文字，则为*或`null`。 *
com.no9.A

此类的父类
com.no9.AA<T>

此类的泛型列表。该列表不包括外部类的泛型参数
T

此类的所有非扩展函数
var com.no9.A<T>.x: kotlin.Int

此类的所有函数
fun com.no9.A<T>.Funtest(): kotlin.Unit
fun com.no9.A<T>.(kotlin.String.)heelo(): kotlin.Unit
```





## 反射获取泛型实参

```kotlin
interface Api {
    fun getUsers(): List<User>
}

class User

abstract class SuperType<T>{
    val typeParameter by lazy {
        this::class.supertypes.first().arguments.first().type
    }

    val typeParameterJava by lazy{
        this.javaClass.genericSuperclass?.safeAs<ParameterizedType>()?.actualTypeArguments?.first()
    }
}

class SubType : SuperType<String>()

fun main() {
    //first返回给定匹配的第一个元素
    Api::class.declaredFunctions.first { it.name == "getUsers" }
        .returnType.arguments.forEach{
        println(it)
    }

    Api::getUsers.returnType.arguments.forEach(::println)

    //通过Java访问  
    Api::class.java.getDeclaredMethod("getUsers").genericReturnType.safeAs<ParameterizedType>()!!.actualTypeArguments.forEach(::println)

    val subType=SubType()
    subType.typeParameter.let(::println)
    subType.typeParameterJava.let(::println)
}

fun <T> Any.safeAs():T?{
    return this as?T
}
```



## 为数据类实现DeepCopy

```kotlin
fun <T : Any> T.deepCopy(): T {
    //如果不是数据类，直接返回
    if (!this::class.isData) {
        return this
    }

    //拿到构造函数
    return this::class.primaryConstructor!!.let {
            primaryConstructor ->
        primaryConstructor.parameters.map { parameter ->
            //转换类型
            //memberProperties 返回非扩展属性中的第一个并将构造函数赋值给其
            //最终value=第一个参数类型的对象
            val value = (this::class as KClass<T>).memberProperties.first {
                it.name == parameter.name
            }.get(this)

            //如果当前类(这里的当前类指的是参数对应的类型，比如说这里如果非基本类型时)是数据类
            if ((parameter.type.classifier as? KClass<*>)?.isData == true) {
                parameter to value?.deepCopy()
            } else {
                parameter to value
            }


         //最终返回一个新的映射map,即返回一个属性值重新组合的map，并调用callBy返回指定的对象
        }.toMap().let(primaryConstructor::callBy)
    }
}

data class A(val name: String)

data class Dep(val a: A)


fun main() {
    val dep = Dep(A("123"))

    val copyDep = dep.copy()
    val deepCopy = dep.deepCopy()

    println(dep === copyDep)
    println(dep === deepCopy)

    println(dep.a === copyDep.a)
    println(dep.a === deepCopy.a)
}
```



## Model映射

```kotlin
data class User(var name: String)


inline fun <reified From : Any, reified To : Any> From.mapAs(): To {
    return From::class.memberProperties.map { it.name to it.get(this) }
        .toMap().mapAs()
}

inline fun <reified To : Any> Map<String, Any?>.mapAs(): To {
    //反射拿到构造函数
    return To::class.primaryConstructor!!.let {
        it.parameters.map { kParameter ->
            //获得对应的value，判断参数是否为null,如果参数允许为null，直接返回null,否则抛出异常
            kParameter to (this[kParameter.name] ?: if (kParameter.type.isMarkedNullable) null
            else throw IllegalArgumentException("${kParameter.name} is required but missing"))
        }.toMap().let(it::callBy)
    }
}


fun main() {
    val user = User("petterp")
    val userVO: User = user.mapAs()
    println(userVO)

    val userMap = mapOf(
        "name" to "petterp"
    )
    val mapAs: User = userMap.mapAs()
    println(mapAs)

}
```



## 可释放对象引用的不可空类型

```kotlin
fun <T : Any> releaseableNotNUll() = ReleasableNotNull<T>()

class ReleasableNotNull<T : Any> : ReadWriteProperty<Any, T> {
    private var value: T? = null
    override fun getValue(thisRef: Any, property: KProperty<*>): T {
        return value ?: throw  IllegalStateException("Not initialized or released already")
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: T) {
        this.value = value
    }

    fun isInitialized() = value != null

    fun release() {
        value = null
    }
}

inline val KProperty0<*>.isInitialized: Boolean
    get() {
        isAccessible = true
        //判断是否初始化
        return (this.getDelegate() as? ReleasableNotNull<*>)?.isInitialized()
            ?: throw IllegalAccessException("Delegate is not an instance of ReleasableNotNull or is null")
    }

fun KProperty0<*>.release() {
    isAccessible = true
    (this.getDelegate() as? ReleasableNotNull<*>)?.release()
}

class Bitmap(var width:Int,var height:Int)

class Activity {
    private var bitmap by releaseableNotNUll<Bitmap>()

    fun onCreate() {
        println(::bitmap.isInitialized)
        bitmap= Bitmap(111,111)
        println(::bitmap.isInitialized)
    }

    fun onDestory() {
        println(::bitmap.release())
    }

    fun sot(){
        println(::bitmap.isInitialized)
    }
}

fun main() {
    val activity=Activity()
    activity.onCreate()
    activity.onDestory()
    //判断对象是为null
    activity.sot()

}
```



## 插件化加载类

