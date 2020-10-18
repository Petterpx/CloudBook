# Kotlin学习之路

### 什么是Kotlin? Kotlin学习指南

```
Kotlin就是一门可以运行在 JAVA虚拟机，Android,浏览器上的静态语言，它与Java100%兼容，如果你对Java非常熟悉，那么你就会发现Kotlin除了自己的标题库之外，大多任然使用经典的Java 集合框架。

学习目标
1. 学会使用 Kotlin
2. 熟悉Java生态
3. 了解一些特性背后的实现

```



---

### Kotlin的数据类型

```
var与val 的区别
var为可变变量,val相当于只读变量，如同java 中的final 一样，val 创建时必须被初始化。
```

1.认识基本类型
2.初步认识类及其相关概念
3.认识区间和数组



#### 基本类型

1. Boolean

   ```
   var a: Boolean=true
   var b: Boolean=false
   
   a为参数
   ```

2. Number

   ```
   var a: Int =8
   var a: Int =0xFF
   ```

3. Float

   ```
   Float类型后面必须加F
   NaN /不是数
   
   val f1: Float =8.0F
   val Positive: Float = Float.POSITIVE_INFINITY	正无穷
   val Negative Float.NEGATIVE_INFINITY			负无穷
   ```

4. Short 短整型

5. Byte 



#### 拆箱装箱与Chart数据类型

```
int anInt = 5
Integer anInteger = 5
在Kotlin 里面，Int实际是 int与Integer的合体，在必要的时候，编译器会自动帮我们进行分辨。
```

---



#### 基础数据类型转换与字符串

```
不可隐式转换
不能直接像Java里一样，将整型赋给 Long，在Kotlin 里,需要显示调用toLong()方法
val anInt : Int =5
val aLong : Long = abInt.toLong()
```

##### 字符串之间的比较

```
== 比较内容，即类似Java 的equals
=== 比较对象是否相同
```

##### 字符串之间的相连

```kotlin
  	//字符串模板 $
  	val a: Int = 1
    val b: Int = 2
    println("" + a + "+" + b + "=" + (a + b))
    println("$a+$b=${a+b}")
    
    如果想在字符串中添加双引号，需要添加转义字符
    Petterp “123”
    var string:String="Petterp \"123\""
    
    需要打印 $ 符号
    var money:String="1000"
    println("$$money")
    
    多行字符串，回车直接可以换行，trimIndent()去除字符串左边为空的位置，这里面无法使用转义
    var raw:String="""
        123
        456
    """.trimIndent()
    println(raw)
```

---



#### 类和对象

1. 类的写法
   - class 类名 {  成员 }
   - 
2. 什么是对象
   - 是一个具体的概念，与类相对
   - 描述某一种类的具体个体
3. 类与对象的关系
   - 一个类通常可以有很多个具体的对象
   - 一个对象本质上只能从属于一个类
   - 即对象与类的关系为 1..n
   - 对象也经常被称为 类的对象 或者 类的实例
4. 类的继承
   - 提取多个类的共性得到一个更抽象的类，即父类
   - 子类拥有父类的一切特征
   - 子类也可以自定义自己的特征
   - 所有类都最终继承自 Any

```kotlin
//一个继承的例子
/*open相当于打开，允许被继承
主构造函数不能包含任何的代码。初始化的代码可以放到以 init 关键字作为前缀的初始化（initializerblocks）中。
在实例初始化期间，初始化块按照它们出现在类体中的顺序执行，与属性初始化器交织在一起：*/
//constructor 主构造函数

class demo1 constructor(a: String, b: String) : Demo(a, b)
class demo2 constructor(a: String, b: String) : Demo(a, b)

open class Demo (a: String,b: String) {
    init {
    	//获取类的名字
        println("${this.javaClass.simpleName} $a $b")
     }
}

fun main(args: Array<String>) {
    var dd1: demo2 = demo2("a", "b")
}

open 覆盖方法也需要加
override 子类覆盖时需要加
```

---



#### 空类型和智能类型转换

1. 空类型
   - //？的意思是，如果为空，则执行前半句，否则执行后半句打印长度
   - //!! 表示强制允许编译器忽略空类型安全
   
```kotlin
   java 代码
   public class MyClass {
       public static void main(String[] a){
           System.out.println(demo().length());
       }
       public static String demo(){
           return null;
       }
   }
   
   Kotlin 代码
   fun demo(): String? {
       //如果加了 ？,编译器就会允许返回null,否则直接报错
       return null
   }
   
   fun main(args: Array<String>) {
       //？的意思是，如果为空，则执行前半句，否则执行后半句打印长度
       println(demo()?.length)
       
       val a: String? = "123"
       //!! 告诉编译器允许编译器忽略空类型安全
       println(a!!.length)
   }
```

2. 安全的类型转换

```kotlin
//父类强转为子类测试
//继承关系，Chaid继承自Parent

java 
public class Test {
    public static void main(String[] args) {
        Parent parent=new Parent();
        ((Chaid) parent).pp();
    }
}

Kotlin
fun main(args: Array<String>) {
    //父类强转为子类测试  
    val parent: Parent = Parent()
    //如果parent转换失败，则返回null,程序而不会崩溃
    val chaid: Chaid? = parent as? Chaid
    //此时打印即为null
    println(chaid)
}
```



---

#### 区间

- 一个数学上的概念，表示范围
- ColsedRange 的子类，IntRanage 最常用
- 基本写法
  - 0..100 表示 [0,100]
  - 0 until 100 表示 [0,100) 
  - i in 1..100 判断 i 是否在区间 [0,100] 中

```kotlin
fun main(args: Array<String>) {
    val range:IntRange=0..1024      //[0,1024]
    val ranege_exclusive:IntRange=0 until 1024  //[0,1024)
    val range_min:IntRange= 0..-1
    
    //判断是不是空
    println(range_min.isEmpty())

    //判断是否包含1024
    println(ranege_exclusive.contains(1024))
    //写法不同，contains 内部也是用的下面这种方法
    println(1024 in range)

    //迭代--相当于输出range数组
    for (i in range){
        print("$i,")
    }
}
```

---



#### 数组的使用方法

在Kotlin里面，基本类型的数组，都是定制的，目的是为了避免不必要的装箱与拆箱，节省效率

- 基本写法

  val array: Array <String> = arrayOf(...)

- 基本操作

  - print array[i]   输出第 i 个成员
  - array[i] = "Hello" 给第i 个成员赋值
  - array.size 数组的长度

```kotlin
val arrayOfInt: IntArray = intArrayOf(1, 3, 5)
val arrayOfChar: CharArray = charArrayOf('H', 'e', 'l', 'l', 'o')
val arrayOfString: Array<String> = arrayOf("Petterp", "p")
fun main(args: Array<String>) {
    //for迭代
    for (int in arrayOfInt) {
        println(int)
    }
    //返回[0,2)区间的元素
    println(arrayOfString.slice(0 until 2))
    //使用分隔符创建一个字符串
    println(arrayOfChar.joinToString(""))
    //将指定数组中的字符转换为字符串
    println(String(arrayOfChar))
    //打印类的全名
    println(Kotlin2::class.java.name)
    //打印类名区间
    println(Kotlin2::class.java.simpleName.slice(0 until 2))
    //基本类型转换
    val hello:Long= arrayOfChar.size.toLong()
    println(hello)
}

class Kotlin2 {

}

```



---

### 程序结构

#### 常量与变量

- val  声明常量，类似于 Java 的final 关键字，不可被重复赋值
- 在Kotlin 里面有类型推导，编译器可以推导量的类型，所以可推导类型定义时可以不用写数据类型，
- 运行期常量 val x=getX()    利用反射手段是可以修改值
- 编译器常量 **const** val x=1  代码在编写过程中已经确定了值，代码中引用到的位置，都替换成了 x，可以在字节码中看到
- var  声明变量 

```kotlin
val data: String = "Petterp"
val data2 = data
//类型推导，编译器已经知道它的类型是 Int
val data3 = 1
fun main(args: Array<String>) {
    println(data)
    println(data2)
    println(data3)
}
```



```kotlin
现在添加 const 

const val data: String = "Petterp"
const val data2 = data
//类型推导，编译器已经知道它的类型是 Int
val data3 = 1
fun main(args: Array<String>) {
    println(data)
    println(data2)
    println(data3)
}
```

所以这就是编译器常量的作用。编译器常量的引用都是直接指明该变量的值



---

#### 函数

```kotlin
//函数写法
fun Demo1(a: Int): Int {
    return a
}
//当函数返回一个表达式的值是，可以使用一下方式
fun Demo2(a: Int) = a

//匿名函数，需要赋值给一个函数变量
val k = fun(a: Int): Int {
    return a
}

fun main(args: Array<String>) {
    println(Demo1(100))
    println(Demo2(200))
    println(k(300))
}
```

注意事项

- 功能要单一
- 函数名要做到顾名思义
- 参数个数不要太多



---

#### Lambda表达式(非常重要)

1. 什么是Lambda表达式

   - Lambda 实际上就是匿名函数
   - 写法 **{ [参数列表]  -> [函数体，最后一行是返回值] }**
   - 举例  **val sum= { a: Int , b: Int -> a+b}**

2. Lambda 类型表示

   - （）-> Unit  无参，返回值为Unit
   - (Int) -> Int    //传入整型，返回一个整型
   - （String， （String）->String）->Boolean  //传入一个String，Lambda表达式，返回Boolean

3. Lambda 表达式的调用

   - 用 () 进行调用

   - 等价于 invoke()

   - ```kotlin
     举例
     val sum{a:Int,b:Int -> a+b}
     
     调用
     sum（1,2）
     sum.invoke(1,2)
     ```

4. Lambda 表达式的简化

   - 函数参数调用时最后一个 Lambda 可以移出去
   - 函数参数只有一个Lambda,调用时小括号可省略
   - Lambda 只有一个参数 可默认为 it
   - 入参，返回值与形参一直的函数可以用函数引用的方式作为实参传入

   ```kotlin
   //Lambda
   //原始写法
   args.forEach ({ println(it) })
   //对于函数来说，如果最后一个参数是Lambda表达式，小括号可以移到外面
   args.forEach(){ println(it) }
   //如果小括号里什么都没有，可以删掉小括号
   args.forEach { println(it)}
   //如果传入的这个函数和需要接收的Lambda表达式类型一样，那么进一步简化
   args.forEach (::println)
   ```



---

#### 类成员(成员方法，成员变量)

1. 什么是类成员

   - 属性：或者说成员变量，类范围内的变量
   - 方法：成员函数，类范围内的函数

2. 函数和方法的区别

   - 函数强调功能本身，不考虑从属
   - 方法的称呼通常是从类的角度出发

3. 定义方法

   ```kotlin
   写法与普通函数完全一致
   class A {
       //简短写法，没有返回值类型的情况下
       fun say(name: String) = println("Hello $name")
       fun phone(phone:String):String{
          return phone
       }
      
   }
   ```

4. 定义属性

   - 构造方法参数中 val / var 修饰的都是属性

   - 类内部也可以定义属性

   - ```kotlin
     // 加修饰的为属性，b只是普通的一个构造方法参数
     class A(val a: Int, b: Int) {
         var b = 0
     }
     ```

5. 属性访问控制

   - 属性可以定义 getSet / setter

   - ```kotlin
     class B{
         //val 修饰的 不可变 无set方法
         val demo:Int=0
      /*   set(value) {
     
         }*/
         var demo2:Int=0
         set(value) {
             println(demo2)
             field = value
         }
     }
     ```

6. 属性初始化

   - 属性的初始化尽量在构造方法中完成

   - 无法在构造方法中初始化，尝试降级为局部变量

   - var 用 lateinit 延迟初始化，val 用 lazy

   - 可空类型谨慎用 null 直接初始化

   - ```K
     class X
     class A(val a: Int, b: Int) {
         var b = 0
         //laterinit 延迟初始化
         lateinit var c: String
         lateinit var d: X
         val e: X by lazy {
             println("Start X")
             X()
         }
         var cc: String? = null	//不推荐这种写法
     }
     ```

---

#### 基本运算符

- 任意类可以定义或者重载父类的基本运算符

- 通过运算符对应的具名函数来定义

- 对于参数个数做要求，对参数和返回值类型不做要求

- 不能像Scala一样定义任意运算符

  ```kotlin
  //使用operator关键字可以重载基本运算符，比如下面的plus函数加上operator，就相当于基本运算中的 +
  //运算符重载要求与运算符的函数名对应，比如要重载加法，函数名就必须是 plus
  
  class Complex(var real:Int,var imaginzry:Int){
      operator fun plus(other:Complex):Complex{
          return Complex(real+other.real,imaginzry+other.imaginzry)
      }
  	
      operator fun plus(other: Int):Complex{
          return Complex(real+other,imaginzry)
      }
      operator fun  plus(other: Any):Int{
         return real.toInt()
      }
  
      override fun toString(): String {
          return "$real+$imaginzry"
      }
  }
  
  class Book{
      //infix 中指表达式,不用点括号去调用
      infix fun on(any:Any):Boolean{
          return false
      }
  }
  class Desk
  
  fun main(args: Array<String>) {
      val c1=Complex(3,4)
      val c2=Complex(1,2)
      println(c1+c2)
      println(c1+5)
      println(c1+"Petterp")
  	
      //-name <Name>
      //in 表示有这个元素返回1 否则返回-1
      if ("-name" in args){
          println(args[args.indexOf("-name")+1])
      }	
     
  
  }
  
  
  ```

---

#### 表达式

1. 中缀表达式

   - 只有一个参数，且用 infix 修饰的函数

   - ```kotlin
     //infix（中缀表达式） 定义的函数不必动过.xx() 的形式调用，而是可以通过函数名(不必写括号) 调用。这种写法在Dsl 中比较常见，在代码中慎用，影响可读性
     class Book{
         //infix 
         infix fun on(any:Any):Boolean{
             return false
         }
     }
     class Desk
     
     
     fun main(args: Array<String>) {
     
         if(Book() on Desk()){
     
         }
     
     }
     ```

2. if表达式

   - if ..else 同java 一样，但是Kotlin 里面具有返回值，所以称为表达式

   - 表达式与完备性 (即在用 if表达式 赋值时，所有条件都必须完整)

   - ```kotlin
     val x=if(b<0) 0 else b
     val x=if(b<0) 0 //错误，赋值时，分支必须完备
     ```

3. when表达式

   - 加强版 switch，支持任意类型

   - 支持春表达式条件分支(类似 if)

   - 表达式与完备性

   - ```  kotlin
     fun main(args: Array<String>) {
         val x=99
         val b=when(x){
             in 1..100 ->  10
             !in  1..100 -> 20
             args[0].toInt() -> 30
             else -> x
         }
         when(x){
             in 1..100 -> println("ok")
             else -> println("no")
         }
     //    val  mode=when{
     //        args.isNotEmpty()&& args[0]=="1" ->1
     //        else ->0
     //    }
         println(b)
     }
     ```



---

#### for循环

```kotlin
fun main(args: Array<String>) {
    for((i,value) in args.withIndex()){
        println("$i  -> $value")
    }

    for(i in args.withIndex()){
        println("${i.index} -> ${i.value}")
    }
}
```



---

#### 异常捕获

1. try...catch

   - catch 分支匹配异常类型

   - 可以写为表达式，用来赋值

   - ```kotlin
     val  a=try {
             100/10
         }catch (e:Exception){
             0
         }
     ```

2. finally

   - finallu 无论代码是否抛出异常都会执行

   - ```kotlin
     //注意下面的寫法finaly还是会先执行，最后才是 return
      fun  P(): Int {
         //try 表达式
         return try {
             100 / 10
         } catch (e: Exception) {
             0
         } finally {
             println("我先打印")
         }
     ```

---



#### 具名参数，变长参数，默认参数

1. 具名参数

   - 给函数的实参附上形参

   - ```kotlin
     //这样写的话，参数的位置就不会产生影响
     
     fun main(args: Array<String>) {
         sum(b=1,a=2)
     }
     
     fun sum(a: Int, b: Int) {
         println("a=$a b=$b")
     }
     ```

2. 变长参数

   - 某个参数可以接受多个值

   - 可以不为最后一个参数

   - 如果传参时有歧义，需要使用具名参数

   - ```kotlin
     fun main(vararg: Array<String>) {
         sum(1, 2, 3, 4, b = "B")
     }
     
     fun sum(vararg a: Int, b: String) {
         a.forEach(::println)
         println(b)
     }
     ```

3. Spread Operator

   - 只支持展开 Array

   - 只用于变长参数列表的实参

   - 不能重载

   - 不能算一般的运算符

   - ```kotlin
     fun main(vararg: Array<String>) {
         sum(1, 2, 3, 4, b = "B")
     
         //Spread Operator,只支持Array
         val array= intArrayOf(1,3,4,5)
         sum(*array,b="B")
     }
     
     fun sum(vararg a: Int, b: String) {
         a.forEach(::println)
         println(b)
     }
     ```

4. 默认参数

   - 为函数参数指定默认值

   - 可以为任意位置的参数指定默认值

   - 传参时，如果有歧义，需要使用具名参数

   - ```kotlin
     fun main(vararg: Array<String>) {
         //调用者未传值，使用的是默认值
         sum(b="B")
         sum(1,"B")
     }
     
     //如果有个函数经常使用一个值，那么在其声明的时候就可以指定这个值
     fun sum(a: Int = 0, b: String) {
         println("a=$a  b=$b")
     }
     ```

---



### 面向对象

#### 抽象类与接口

1. 什么是接口

   - 接口，直观理解就是一种约定

   - ```kotlin
     interface A {
         fun Print()
     }
     
     //接口继承
     interface A1 : A {
         //也可以定义一个变量，这里实际相当于方法,但是无法有set，get方法
         var a: Int
         fun ko() {
             println(a)
         }
     }
     
     class TestA(override var a: Int) : A1 {
         override fun Print() {
             println("Petterp")
         }
     }
     
     fun main() {
         val test = TestA(1)
         test.Print()
         test.ko()
     }
     ```

2. 接口

   - 接口不能有状态
   - 必须由类对其进行实现后使用

3. 抽象类

   - 实现了一部分协议的半成品

   - 可以有状态，可以有方法实现

   - 必须由子类继承后使用

   - ```kotlin
     interface P1
     
     interface P2 : P1
     
     abstract class A {
         //打印出类名
         fun Print() = println(javaClass.simpleName)
     }
     
     class B : A(), P2
     
     fun main() {
         //不可以这样用，因为没有子类继承
     //    val a = A
     
         //关系可以这样来写
         val p1: P1 = B()
         val p2: P2 = B()
         val a: A = B()
         val b: B = B()
         b.Print()
     }
     ```

4. 抽象类与接口的共性

   - 比较抽象，不能直接实例化
   - 有需要子类(实现类) 实现的方法
   - 父类 (接口) 变量可以接受子类 (实现类) 的实例赋值

5. 抽象类和接口的区别

   - 抽象类有状态，接口没有状态
   - 抽象类有方法实现，接口只能有无状态的默认实现
   - 抽象类只能单继承，接口可以多实现
   - 抽象类反映本质，接口体现能力



---

#### 继承

- 父类需要open 才可以被继承

- 父类方法，属性需要open 才可以被覆写

- 接口，接口方法，抽象类默认为 open

- 覆写父类 (接口) 成员 需要 onverride 关键字

- ```
  class D:A() , B ,C
  ```

- 注意继承类时实际上调用了父类构造方法

- 类只能单继承，接口可以多实现

- ```kotlin
  //继承与子类重写父类的Demo
  
  abstract class Person(open val age: Int) {
      abstract fun work()
  }
  
  class MaNong(age: Int) : Person(age) {
      //重写属性
      override val age:Int
      get() = 0
      override fun work() {
          println("我是碼農")
      }
  }
  
  class Doctor(age: Int) : Person(age) {
      override fun work() {
          println("我是醫生")
      }
  }
  
  fun main() {
      val manong = MaNong(20)
      val doctor = Doctor(100)
      println(manong.age)
      println(doctor.age)
  }
  ```




- 接口代理

- 接口方法实现交给代理类实现

- ```kotlin
  class Manager(driver:Driver):Driver by driver
  
  //接口代理的Demo
  
  interface AA{
      fun Print1()
  }
  interface BB{
      fun Print2()
  }
  
  class Car:AA{
      override fun Print1() {
          println("AA")
      }
  }
  
  class Bar:BB{
      override fun Print2() {
          println("BB")
      }
  }
  //
  //class AB:AA,BB{
  //    override fun Print1() {
  //        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
  //    }
  //
  //    override fun Print2() {
  //        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
  //    }
  //
  //}
  
  //by 接口代理
  class SendWill(val aa:AA,val bb:BB):AA by aa,BB by bb
  
  fun main() {
      val aa=Car()
      val bb=Bar()
      val send=SendWill(aa,bb)
      send.Print1()
      send.Print2()
  }
  ```



- 接口方法冲突

- 接口方法可以有默认实现

- 签名一致且返回值相同的冲突

- 子类(实现类) 必须覆写冲突方法

- ```
  super <[父类 (接口)  名 ]>.[方法名]([参数列表]
  ```

- ```kotlin
  Demo
  interface A {
      fun setT() = "接口A"
  }
  
  interface B {
      fun setT() = "接口B"
  }
  
  abstract class C {
      open fun setT() = "抽象类C"
  }
  
  class Demo(val a: Int) : A, B, C() {
      override fun setT(): String {
          return when (a) {
              1 -> super<A>.setT()
              2 -> super<B>.setT()
              3 -> super<C>.setT()
              else -> ""
          }
      }
  }
  
  fun main() {
      println(Demo(1).setT())
      println(Demo(2).setT())
  }
  ```



---

#### 类及其成员可见性

![1550315847886](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1550315847886.png)



#### Object(最简单的单例模式)

- 只有一个实例的类

- 不能自定义构造方法

- 可以实现接口，继承父类

- 本质上就是单例模式最基本的实现

- ```java
  Koltin--
  interface A {
      fun a1()
  }
  
  abstract class C
  
  object KotlinDan : C(), A {
      override fun a1() {
          println("Kotlin")
      }
  }
  
  
  Java--
  public class Kotlin_java {
      public static void main(String[] args) {
          //引用Kotlin代码
          KotlinDan.INSTANCE.a1();
          //在java里面
          JavaDan.inter.Ag();
      }
  }
  
  //java中的简易单例例子
   class JavaDan{
      static JavaDan inter=new JavaDan();
      private JavaDan(){}
      void Ag(){
          System.out.println("Java");
      }
   }
  ```

---

#### 伴生对象与静态成员

- 每个类可以对应一个伴生对象

- 伴生对象的成员全局独一份(对于类来说)

- 伴生对象的成员类似 Java 的静态成员

- 在Kotlin中，静态成员考虑用包级函数，包级变量替代

- JvmField 和 JvmStatic 的使用

- ```kotlin
  Kotlin--
  class Demo private constructor(val value:Double){
      //伴生对象
      companion object {
          @JvmStatic
          fun A(double: Double): Demo {
              return Demo(double)
          }
          @JvmField
          val TAG:String="Petterp"
      }
  }
  
  fun main() {
      println(Demo.A(1.1).value)
  }
  
  Java--
  
  public class KotlinJava {
      public static void main(String[] args) {
          //java代码，未添加 @JvmStataic 与 JvmField
          Demo demo = Demo.Companion.A(1.1);
          String tag = Demo.Companion.getTAG();
  
          //添加后
          Demo demo1 = Demo.A(1.1);
          String  tag1=Demo.TAG;
      }
  }
  
  ```





---

#### 方法重载与默认参数

1. 方法重载

   - Overloads

   - 名称相同，参数不同的方法

   - Jvm函数签名的概念 ： 函数名，参数列表

   - 跟返回值没有关系

   - ```kotlin
     class A{
         fun a():Int{
             return 0
         }
         fun a(int: Int):Int{
             return int
         }
     
         //方法重载与返回值无关
        /* fun a():String{
     
         }*/
     }
     
     fun main() {
         val a=A().a()
         val a2=A().a(123)
     
     }
     ```

2. 默认参数

   - 为函数参数设定一个默认值

   - 可以为任意位置的参数设置默认值

   - 函数调用产生混淆时用具名参数

   - ```kotlin
     Kotlin--
     class A {
     //    fun a():Int{
     //        return 0
     //    }
     //    fun a(int: Int):Int{
     //        return int
     //    }
     
         //使用具名参数替代上面的重载方法
         @JvmOverloads	
         fun a(int: Int = 0): Int {
             return int
         }
     }
     
     fun main() {
         val a = A().a()
         val a2 = A().a(123)
     }
     
     Java调用时--
     public class KotlinJava {
         public static void main(String[] args) {
             A a=new A();
             //在Kotlin代码上添加 @JvmOverloads 就可以让java识别具名参数
             a.a();
             a.a(123);
         }
     }
     ```

3. 方法重载与默认参数的关系

   - 二者的相关性以及 @JvmOverloads

   - 避免定义关系不大的重载

   - 不好的设计比如：

     ```kotlin
     Java代码中的一个Bug
     public class Bug {
         public static void main(String[] args) {
             List<Integer> list=new ArrayList<>();
             list.add(1);
             list.add(2);
             list.add(10);
             list.add(20);
             System.out.println(list);
             //异常，数组越界
             list.remove(10);
             list.remove(2);
         }
     }
     
     在Kotlin代码中
     fun main() {
         val list = ArrayList<Int>()
         list.add(1)
         list.add(2)
         list.add(10)
         list.add(20)
         println(list)
         list.removeAt(10)
         list.remove(2)
     }
     ```



---

#### 扩展成员

- 为现有类添加方法，属性

- ```
  fun X.y():Z {...}
  val X.m 注意扩展属性不能初始化，类似接口属性
  ```

- Java 调用扩展成员类似调用静态方法

- ```kotlin
  operator fun String.times(int: Int):String{
      val stringBuilder=StringBuilder()
      for (i in 0 until  int){
          stringBuilder.append(this)
      }
      return stringBuilder.toString()
  }
  
  val String.op:String
      get() = "123"
  
  fun main() {
      println("abc".times(16))
      //扩展方法前加了operator，重载运算符
      println("abc"*16)
      
      println("abc".op)
  }
  
  
  java调用
  public class A {
      public static void main(String[] args) {
          System.out.println(Kotlin2Kt.times("abc",16));
          System.out.println(Kotlin2Kt.getOp("Petterp"));
      }
  }
  
  ```





---

#### 属性代理

- 定义方法

  ```kotlin
  val/var <property name>: <Type> by <experession> 
  //翻译  property-属性  type-类型 experssion-函数调用
  ```

- 代理者需要实现相应的 setValue/getValue 方法

- ```kotlin
  // Demo--
  class A{
      val p1:String by lazy {
          "123"
      }
      //属性代理，实际调用getValue
      val p2 by X()
      //需要实现setValue
      var p3 by X()
  }
  class X{
      private var value: String? =null
      operator fun getValue(thisRef:Any?,property:KProperty<*>):String{
         return value?:""
      }
      operator fun setValue(thisRef: Any?,property: KProperty<*>,string: String){
          this.value=string
      }
  }
  
  fun main() {
      val a=A()
      println(a.p1)
      println(a.p2)
      //此时为空，因为未设置值
      println(a.p3)
      a.p3="123"
      println(a.p3)
  }
  ```





---

#### 数据类

- 再见，JavaBean

- 默认实现的copy,toString 等方法

- componentN 方法

- allOpen 和 noArg 插件 （设计的角度来说，data class不允许有子类，所以如果你要改写的话，需要用到这两个插件，会在编译期通过修改字节码的方式去掉 final关键字，并增加一个无参构造方法）

- data class 初始化的时候，一定要给它的属性赋值(带参数)，即它并没有默认的无参构造方法

- ```kotlin
  //加了data 之后，自动实现各种方法，可查看字节码发现
  data class A(val id:Int,val name:String)
  class ComponentX{
      //手动实现component1()
      operator fun component1():String{
          return "PP"
      }
      operator fun component2():Int{
          return 0
      }
  }
  fun main() {
      println(A(1,"Petterp"))
      val (a,b)=ComponentX()
      println("a=$a,b=$b")
      //打印结果
      /*A(id=1, name=Petterp)
      a=PP,b=0*/
  }
  ```

  ```
  allOpen noArg 配置方法
  
  ```

---

#### 内部类(this@Outter,this@Inner)

1. 内部类

   - 定义在类内部的类

   - 与类成员有相似的访问控制

   - 默认是静态内部类，非静态用 inner 关键字

   - this@Outter , this@Inner 的用法 

   - 如果内部类依赖外部类，那么使用非静态内部类，否则反之

   - ```kotlin
     class A {
         var int:Int = 5
     
         //加了inner 为非静态内部类
         inner class A1 {
             var int:Int = 10
             fun Print(){
                 //内部类的变量
                 println(int)
                 //外部变量
                 println(this@A.int)
             }
         }
     }
     ```

2. 匿名内部类

   - 没有定义名字的内部类

   - 类名编译时生成，类似 Outter$1.class   (看起来没有名字，实际编译时有自己的Id)

   - 可继承父类，实现多个接口，与Java 注意区别

     ```kotlin
     interface OnclickListener {
         fun onclick()
     }
     
     class View {
         var onclickListener: OnclickListener? = null
     }
     
     open class A
     
     fun main() {
         val view=View()
         view.onclickListener=object : A(),OnclickListener{
             override fun onclick() {
                 
             }
         }
     }
     ```



---

#### 枚举

- 实例可数的类，注意枚举也是类

- 可以修改构造，添加成员

- 可以提升代码的表现力，也有一定的性能开销

- ```kotlin
  //枚举类也是有构造方法的，我们可以在它的构造方法中传入参数
  enum class LogLevel(val id:Int){
      A(1),B(2),C(3),D(4),E(5) ;
      //需要分号隔开
      fun getTag():String{
          return "$id,$name"
      }
  	
      override fun toString(): String {
          return "$name,$ordinal"
      }
  }
  
  //LogLevel 更像是 LogLevel2的语法糖
  //它们两个是等价的
  class LogLevel2(val id:Int) private constructor(){
      //伴生对象写法
      companion object {
          val A=LogLevel2(1)
          val B=LogLevel2(2)
          val C=LogLevel2(3)
          val D=LogLevel2(4)
          val E=LogLevel2(5)
      }
  
      constructor(i: Int) : this()
  }
  
  fun main() {
      println("${LogLevel.A},${LogLevel.A.ordinal}")
      println(LogLevel.A.getTag())
      //遍历枚举，重写了toString()方法
      LogLevel.values().map(::println)
  
      //返回"A"的实例
      println(LogLevel.valueOf("A"))
  }
  ```

---

#### 密封类(sealed Class)

- 子类可数  (枚举是实例可数)

- 要注意密封类与枚举的不同，看以下Demo

- ```kotlin
  //在以下Demo中，这是一个音乐播放Demo
  //需要不同指令及不需要参数的地方我们可以用枚举实现，而那些需要不同指令参数的地方我们用枚举就无法实现了
  
  //sealed的子类只能继承在与Sealed同一个文件当中，或者作为它的内部类
  sealed class PlayerCmd {
      class Play(val yrl: String, val postition: Long = 0) : PlayerCmd()
  
      class Seek(val postition: Long) : PlayerCmd()
  
      object Pause : PlayerCmd()
  
      object Resume : PlayerCmd()
  
      object Stop : PlayerCmd()
  }
  
  enum class PlayerState{
      IDLE,PAUSE,PLAYING
  }
  ```





### 高阶函数

#### 高阶函数的基本概念

- 传入或者返回函数的函数

- 函数引用  ::println

- 带有 Receiver 的引用， pdfPrinter :: println

- ```kotlin
  
  fun main(args: Array<String>) {
      //包级函数引用
      args.forEach(::println)
  
      //类引用
      val helloWorld = Hello::world
  //    args.filter(String::isNotEmpty)
  
      //调用者引用方法
      args.forEach(PdfPrinter()::println)
  
      //Demo1
      println(demo("k",::getRest))
      //Demo2
      demo2(Hello()::world)
  }
  
  class Hello {
      fun world(){
          println("123")
      }
  }
  
  class PdfPrinter {
      fun println(any: Any) {
          kotlin.io.println(any)
      }
  }
  
  fun getRest(s1:String):String="Petterp $s1"
  
  fun demo(a: String, method: (s1: String) -> String): String = method(a)
  
  
  fun demo2(methoud2:()->Unit)=methoud2()
  ```



---

#### 常用的高阶函数

- forEach (通常用于遍历集合)

  ```Kotlin
   val list = listOf(1, 3, 4, 5, 6, 7, 8)
      val newList=ArrayList<Int>()
      list.forEach {
          newList.add(it*2)
      }
      newList.forEach(::println)
  ```

- map (用于集合的映射，还可以用于集合转换)

  ```kotlin
    val list = listOf(1, 3, 4, 5, 6, 7, 8)
    val newList = list.map { it * 2 + 3 }
    newList.forEach(::println)
  ```

- flatmap (用于吧集合的集合扁平化成集合，换可以结合map进行一些变换)

  ```kotlin
  fun main() {
    .flatMap { intRange ->
          intRange.map { intElement ->
              "No.$intElement"
          }
      }.forEach(::println)
      
      //简写
      list.flatMap { 
          it.map { "No.$it" }
      }.forEach(::println)
  }
  ```
  
- fold （用于求和并加上一个初始值因为fold不同于map,fold对初始值没有严格限制，因此fold还可以进行类型变换）

  ```kotlin
  fun main() {
      //遍历0..6的阶乘
      (0..6).map(::factorial).forEach(::println)
  
      //加初始值5     结果： 879
      println((0..6).map(::factorial).fold(5){acc, i ->acc+i})
      //加上psotion
      println((0..6).map(::factorial).foldIndexed(StringBuilder()){postion,acc, i ->acc.append("$i-$postion,")})
      //更改返回类型      结果  1,1,2,6,24,120,720,
      println((0..6).map(::factorial).fold(StringBuilder()){acc, i -> acc.append("$i,") })
      //倒序
      println((0..6).map(::factorial).foldRight(StringBuilder()){i,acc -> acc.append(i).append(",") })
      
      //字符串连接
      println((0..6).joinToString(","))
  }
  
  //求阶乘
  fun factorial(n:Int):Int{
      if (n==0){
          return 1
      }
      return (1..n).reduce { acc, i -> acc*i}
  }
  ```

- reduce (求和)

  ```kotlin
  fun main() {
      val list = listOf(
          1..20,
          6..100,
          200..220
      )
      //求和  结果：9655
      println(list.flatMap { it }.reduce { acc, i -> acc + i })
  }
  ```

- filter(用于过滤，如果传入的表达式值为true，就保留)

  ```kotlin
   val list = listOf(
         1,3,2
      )
      //传入的表达式值为true就保留
      println(list.filter { it%2==0 })
      //满足条件的保留，并有他们的位置
      println(list.filterIndexed{index, i ->  i%2==1 })
  ```

- take (通常为带有条件的循环遍历)

  ```kotlin
    val list = listOf(
         1,3,2
      )
    	//从第一个元素开始，只返回符合的元素，遇到不符合停止
      println(list.takeWhile { it%2==1 })
      //从最后一个元素开始，一直取到不符合的就返回前面的元素
      println(list.takeLastWhile { it%2==0})
      //返回最后一个元素到指定元素位置的列表,不包含指定位置元素
      println(list.takeLast(4))
      //返回第一个一个元素到指定元素位置的列表,不包含指定位置元素
      println(list.take(4))
      //参数是个方法，返回值是一个布尔类型，为真返回对象T,否则返回null
      println(list.takeIf { it.size>6 })
  ```

- let,apply,with,use（用于简化代码，use可以简化 colse,try/catch 统一使用模板）

  ```kotlin
  data class Person(val name: String, val age: Int) {
      fun work() {
          println("我是$name")
      }
  }
  
  fun main() {
      findPerson()?.let {
          it.work()
          println(it.age)
      }
  
      //apply相当于一个扩展方法
      findPerson()?.apply {
          work()
          println(age)
      }
  
      BufferedReader(FileReader("A.txt")).use {
          var line:String?
          while (true){
              line=it.readLine()?:break
              println(line)
          }
      }
  
  //    val br=BufferedReader(FileReader("hello.txt")).readLine()
  }
  
  fun findPerson(): Person? {
      return null
  }
  ```

---

#### 尾递归优化

> 尾递归指的是函数调用自己之后没有任何操作，也就是一个函数中所有递归形式的调用都出现在函数的末尾。尾递归可以使用 tailrec 关键字优化

- 递归的一种特殊形式

- 调用自身后无其他操作

- talrec关键字 提示编译器尾递归优化

- 尾递归可以直接转换为迭代

- ```kotlin
  data class ListNode(val value:Int,var next:ListNode?=null)
  
   tailrec fun findListNode(head:ListNode?,value:Int):ListNode?{
      head?:return null
      if (head.value==value) return head
      return findListNode(head.next,value)
  }
  
  
  fun main() {
      val Max_Node_Court=1000000
      val head=ListNode(0)
      var p=head
      for (i in 1..Max_Node_Court){
          p.next= ListNode(i)
          p=p.next!!
      }
      println(findListNode(head,Max_Node_Court-2)?.value)
  }
  
   fun factorial(n:Long):Long{
      return n* factorial(n-1)
  }
  
  data class TreeNode(val value:Int){
      var left:TreeNode?=null
      var right:TreeNode?=null
  }
  
   fun findTreNode(root:TreeNode?,value: Int):TreeNode?{
      root?:return  null
      if (root.value==value) return root
      return findTreNode(root.left,value)?:return findTreNode(root.right,value)
  }
  ```

---

#### 闭包

- 函数运行的环境

- 它持有函数运行的状态

- 函数内部可以定义函数

- 函数内部也可以定义类

- ```kotlin
  val String="HelloWord"
  
  //Lambda 返回一个无参函数
  fun makeFun():() -> Unit{
      var count=0
      return fun (){
          println(++count)
      }
  }
  
  fun main(args: Array<String>) {
  //    val x= makeFun()
  //    x()
  //    x()
      val add5= add(5)
      println(add5(2))
  
      for (i in fibonac()){
          if (i>100) break
          println(i)
      }
  }
  
  
  fun fibonac():Iterable<Long>{
      var first=0L
      var second=1L
      return Iterable {
          object :LongIterator(){
              override fun hasNext()=true
              override fun nextLong(): Long {
                      val result=second
                      second+=first
                      first=second-first
                      return result
              }
  
          }
      }
  }
  
  //Demo2
  //简写
  fun add(x:Int)=fun (y:Int)=x+y
  //完整
  fun add2(x: Int):(Int)->Int{
      return fun (y:Int):Int{
          return x+y
      }
  }
  
  ```



---

#### 函数复合

- f(g(x))

- ```kotlin
  
  val add5={i:Int -> i+5}     //f(g(x))
  val mulyiplyBy2={i:Int ->i*2} // m(x)=f(g(x))
  
  fun main(args: Array<String>) {
  
      println(mulyiplyBy2(add5(8)))       //(5+8)*2
  
      val k= add5 andThen mulyiplyBy2
      println(k(8))
  }
  
  
  
   /*P1,P2 表示参数值，R表示返回值
      addThen为扩展方法
      infix中缀表达式
      Function<P1,P2>  p1为参数类型，P2为返回值类型
  */
  infix  fun <P1,P2,R > Function1<P1,P2>.andThen(function: Function1<P2,R>):Function1<P1,R>{
      return fun (p1:P1):R{
          //返回了一个函数，函数里面又把function这个参数调用了一遍
          //调用的时候又把自己调用了一遍，把自己的返回值传给function
          return function.invoke(this.invoke(p1))
      }
  }
  
  infix  fun <P1,P2,R> Function1<P1,P2>.compose(function: Function1<P2,R>):Function1<P1,R>{
      return fun (p1:P1):R
      {
          return function.invoke(this.invoke(p1))
      }
  }
  ```



---

#### 柯里化

- Currying 简单来说就是多元函数变换成一元函数调用链

- ```kotlin
  //由多参数变换为单参数的变化
  fun Test(a: Int): (String) -> (Int) -> Boolean {
      return fun(b: String): (c: Int) -> Boolean {
          return fun(c: Int): Boolean {
              return true
          }
      }
  }
  
  fun log(tag: String, target: OutputStream, message: Any?) {
      target.write("[$tag] $message\n".toByteArray())
  }
  
  //柯里化
  fun log2(tag: String)
          = fun(target: OutputStream)
          = fun(message: Any?)
          = target.write("[$tag] $message\n".toByteArray())
  
  fun main() {
      log("benny",System.out,"Heelo2wor")
      log2("Petterp")(System.out)("Demo")
  
      ::log.curried()("Petterp")(System.out)("Demo")
  }
  
   fun <P1,P2,P3,R> Function3<P1,P2,P3,R>.curried()
      =fun(p1:P1)=fun (p2:P2)=fun (p3:P3)=this(p1,p2,p3)
  ```



---

#### 偏函数

- 传入部分参数得到的新函数

- 对于某些传值比较固定的参数，偏函数可以将其绑定，然后生成新的函数，而新的函数只需要给除已绑定的参数之外的参数传值，当然你也可以视同 默认参数+具名参数 的方式来实现参数的固定，如果需要固定的参数在中间，虽然说可以通过具名参数来解决，但是很尴尬，因为必须使用一大推具名参数，因此偏函数就诞生了

- ```kotlin
  val test=fun(name:String,love:String):String{
      return "${name}爱$love"
  }
  
  fun <P1,P2,R> Function2<P1,P2,R>.partial1(p1:P1)=fun(p2:P2)=this(p1,p2)
  fun <P1,P2,R> Function2<P1,P2,R>.partial2(p2:P2)=fun (p1:P1)=this(p1,p2)
  
  fun main() {
      val t= test.partial1("Petterp")
      println(t("写Bug"))
  
      val t2= test.partial2("改Bug")
      println(t2("Petterp"))
  }
  ```

---

### 泛型

#### 泛型的基本语法

- 泛华的类型或者说类型的抽象
- 鸭子类型是动态类型和静态语言的一种对象推断分格，在鸭子类型中，关注的不是对象的类型本身，而是他是如何使用的，也就是说我们只关注它的行为。

### java与Kotlin 互操作

#### 基本互操作

1. 空安全类型

   - Kotlin空安全类型原理

     ```
     Kotlin在编译的时候，会增加一个函数调用，会对参数类型，返回值类型进行是否为null的检查
     ```

   - 平台类型 PlatFromType

     ```
     因为Java里面并没有空安全类型，所以可能会出现平台类型的问题，这时候就需要我们开发者自己明白需要使用的参数是否可以为null
     ```

   - @Nullable 和 @NotNull

     ```
     在开发Java代码的时候，可以通过注解的方式来弥补这一点
     ```

2. 几类函数的调用

   - 包级函数：静态方法

     ```
     在java里并没有这种函数，它在编译的时候，会为Kotlin生成一个类，这个类包含了所有包级函数，在java看来，这些都只是静态方法，所以在java调用的时候，按照静态按方法调用即可
     ```

   - 扩展方法：带 Receiver 的静态方法

     ```
     扩展方法只是增加了一个 Receiver 作为参数
     ```

   - 运算符重载：带Receiver 的对应名称的静态方法

3. 常用注解的使用

   - @JvmField : 将属性编译为 JAVA变量

   - @JvmStataic ：将对象的方法编译成 Java静态方法

   - @JvmOverloads : 默认参数生成重载方法

     ```
     如果一个参数带有默认参数，Java实际是看不见的
     ```

   - @file: JvmName : 指定Kotlin 文件编译后的类名

4. NoArg 与 AllOpen

   - NoArg 为被标注的类生成无参构造

     ```
     支持Jpa注解，如 @Entity
     ```

   - AllOpen 为被标注的类去掉final,允许被继承

     ```
     支持 Spring注解，如 @Component
     ```

   - 支持自定义注解类型，例如 @Poko

5. 泛型

   - 通配符 Kotlin 的 * 对应java的 ？

   - 协变与逆变 out/in

     ```
     ArrayList<out String>
     ```

   - 没有Raw 类型

     ```
     java 的List-> 		Kotlin的List<*>
     ```







<br/>



# Kotlin-重构篇-更加接近实际应用

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

# Kotlin-内置类型

## 基本类型

![image-20191228150348866](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vjty36j31gx0u042e.jpg)

```
var b: String = "asdasd"
val c: Int = 15
```



字符串比较，

==,===

== 比较内容是否相等

=== 比较对象是否相等



## 数组

![image-20191228155051782](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vefbhmj31f10u0djk.jpg)

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

![ image-20191228170007121](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vfrkg7j31cy0u0win.jpg) 

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

![image-20191228180401420](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vo2druj30tt05e0u5.jpg)

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



## Nothing

在你的判断逻辑中，充当永远不可能调用的哪一项，比如你有一个when的选择语句，就可以使用Nothing作为你的else返回。

先看一下Nothing是什么

```kotlin
/**
 * Nothing has no instances. You can use Nothing to represent "a value that never exists": for example,
 * if a function has the return type of Nothing, it means that it never returns (always throws an exception).
 */
//*没有实例。您可以使用Nothing来表示“一个永不存在的值”：例如，*如果函数的返回类型为Nothing，则表示该函数永不返回（总是引发异常）。
public class Nothing private constructor()
```

### 示例如下：

有下面这样一段代码：

```kotlin
fun test(type: String) =
    when (type) {
        "key1" -> "条件1"
        "key2" -> "条件2"
        else -> caseDefault()
    }

/** 一个报错的逻辑，为了便于编译器识别，返回类型依然为String
 * 如果是其他情况，相应返回类型需要更改方可便于编译器，否则相应的异常每个方法都需要手动添加 */
private fun caseDefault(): String {
    throw RuntimeException("123")
}
```

使用 **Nothing** 优化这段代码

```kotlin
fun test(type: String) =
        when (type) {
            "key1" -> "条件1"
            "key2" -> "条件2"
            else -> doNothing()
        }

private fun doNothing(message: String="Nothing is all"): Nothing {
    throw RuntimeException(message)
}
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





### 简易计算器

```kotlin
fun main() {
    val res: String = readLine()!!
    if (res.isEmpty()) {
        throw RuntimeException("不可为null")
    }

    val operations = mapOf(
        "+" to ::plus,
        "-" to ::minus,
        "*" to ::times,
        "/" to ::div
    )

    when {
        "+" in res -> {
            val (a, b) = res.split("+")
            println(operations["+"]?.invoke(a.toInt(), b.toInt()))
        }
        "-" in res -> {

        }

        "*" in res -> {

        }
        "/" in res -> {

        }
    }
}

fun plus(a: Int, b: Int): Int {
    return a + b
}

fun minus(a: Int, b: Int): Int {
    return a + b
}

fun times(a: Int, b: Int): Int {
    return a + b
}

fun div(a: Int, b: Int): Int {
    return a + b
}
```





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

![image-20191231173432841](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vg1hydj30f404jjrg.jpg)



如果我们想避免Java直接访问到我们的代码，可以加入以下小技巧,这样当Java调用时就会因不规范而报错。

![image-20191231173528846](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vkpdq7j30ai03paa0.jpg)



![image-20191231173552689](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vk9qtdj30dw03b0sr.jpg)

  

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
-  密封类的子类定义在自身相同的文件中
-  密封类的子类个数有限

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

![image-20191230164458970](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vome3dj30lo05j3yv.jpg)



#### map

```kotlin
 mapOf("Kotlin核心技术" to 99.9,"Androidxxx" to 88).map {
        println("${it.key} --- ${it.value}")
    }
```

![image-20191230165026220](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vghfhnj30ud05zwf7.jpg)





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

![image-20191230231208752](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vln4q7j30s803yaae.jpg)



### Reduce

```kotlin
println(listOf(1, 2, 3).reduce { acc, i ->
        acc + i
    })
```

![image-20191230231732952](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8voxhkvj30tr068wf8.jpg)



### sum

```kotlin
println(listOf(1, 2, 3).sum())
```

![image-20191230231839186](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vmn1qvj30d804x3yl.jpg)



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

![image-20191230233028135](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vlbzekj30pg03cwem.jpg)



//更多集合相关操作，参考https://www.jianshu.com/p/d627bf3d5d72



## SAM

1.  Java：一个参数类型为 **只有一个方法的接口** 的方法
2.  一个参数类型为 **只有一个方法的Java接口** 的java方法

### Sam转换支持对比

|                | **Java**   | **Kotlin** |
| -------------- | ---------- | :--------- |
| **Java接口**   | **支持**   | **支持**   |
| **Kotlin接口** | **支持**   | **不支持** |
| **Java方法**   | **支持**   | **支持**   |
| **Kotlin函数** | **支持**   | **不支持** |
| **抽象类**     | **不支持** | **不支持** |





## Demo练习

### 统计字符个数

```kotlin
File("kotlin.iml").readText().toCharArray().filter { !it.isWhitespace() }
        .groupBy { it }.map {
            it.key+"="+it.value.size
        }.forEach(::println)
```

```
<=12
?=2
x=2
m=9
l=18
v=4
e=43
...
```



### groupBy

分组

![image-20191231112839615](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vhwbplj30qu034glu.jpg)

![image-20191231113027624](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vqivl0j30z905qgm9.jpg)





# Kotlin-泛型

对一个Android开发者来说，不会使用泛型，简直是一种莫大的耻辱，而大多数人对于泛型的使用也仅仅停留在基本的<T>层次。下面，当我们开始学习Kotlin泛型的时候，建议认真去回顾一下Java的泛型，泛型擦除等相关，再来学习Kotlin泛型



## Java泛型

- ？extends   传递的类型必须继承指定的类，并且不支持后期修改

- ?  super  将父类类型赋值给子类泛型声明



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
-  消费者 Consumber<Base> 兼容 Consumer<子类>
-  存在逆变点的类的泛型参数必须声明为 逆变或不变



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

![image-20200105182055659](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vh3xhzj30hk03ndg0.jpg)



### 简单理解

![image-20200105182156070](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vhgzgqj32840s077o.jpg)





## * 星投影

-    可用在变量类型声明的位置
-    可用以描述一个未知的类型
-    所替换的类型在：
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

![image-20200105222110821](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vpxf1mj30r0065mxv.jpg)

报错已解决，原因是getValue方法中的thisRef参数，也就是当前属性对象需要null处理。加上?即可.

新问题todo:为什么在类中无需添加?，而在类中定义一个方法，然后继续使用 局部属性代理 依然报错，不太明白？

> 一些基础 
>
> operator fun <T : AbsModel> getValue(thisRef: Any?, property: KProperty<*>): T {
>   return Models.run {
>       //capitalize首字母大写
>       property.name.capitalize().get()
>   }
> }
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





# Kotlin-注解

> - 注解类似于一个标签
> - 注解信息可用于源码级，编译期，运行时

## 基本

### 限定标注对象

```kotlin
@Target(AnnotationTarget.CLASS)
annotation class Api
```

### 明确注解的标注对象

```kotlin
//标注文件
@file:JvmName("Hello")
```



##常见内置注解的使用

- **Kotlin.annotation.* 用于标注注解的注解**
- **kotlin.* 标准款的一些通用用途的注解**
- **kotlin.jvm.* 用于与java 虚拟机交互的注解**



### 标准库的通用注解

- **Metadata**  kotlin反射的信息通过该注解附带在元素上
- **UnsafeVariance**  泛型用来破除型变限制
- **Suppress**  用来去除编译器警告，警告类型作为参数传入

### Java 虚拟机相关注解

- **JvmField** 免get,set
- **JvmName**  指定类，函数等生成的Jvm名字
- **JvmOverloads**  函数默认参数生成函数重载
- **JvmStatic**  生成静态成员
- **Synchronized** 标记函数为同步函数
- **Throws** 标记函数抛出的异常类型
- **Volatile**  可见性，重排序，禁止优化



## 仿Retrofit 反射读取注解请求网络

Todo



## 注解处理器实现Model映射

####   新建注解model：

```kotlin
//运行时
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS)
annotation class ModelMap
```

#### 创建编译器model: compiler

**build.gradle如下**

```groovy
apply plugin: 'java-library'
apply plugin: 'kotlin'
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.squareup:kotlinpoet:1.4.4'
    implementation project(path: ':annotations')
    implementation 'com.bennyhuo.aptutils:aptutils:1.7.1'
}

sourceCompatibility = "8"
targetCompatibility = "8"

```



#### 创建你的编译类ModelMapProcessor

```kotlin
import com.android.anotionns.ModelMap
import com.bennyhuo.aptutils.AptContext
import com.bennyhuo.aptutils.types.asKotlinTypeName
import com.bennyhuo.aptutils.types.packageName
import com.bennyhuo.aptutils.types.simpleName
import com.bennyhuo.aptutils.utils.writeToFile
import com.squareup.kotlinpoet.*
import javax.lang.model.element.ExecutableElement
import javax.lang.model.element.TypeElement
import com.squareup.kotlinpoet.ParameterizedTypeName.Companion.parameterizedBy
import javax.annotation.processing.*
import javax.lang.model.SourceVersion


/**
 * Created by Petterp
 * on 2020-01-07
 * Function:
 */
@SupportedAnnotationTypes("com.android.anotionns.ModelMap")
@SupportedSourceVersion(SourceVersion.RELEASE_7 )
class ModelMapProcessor : AbstractProcessor() {

    override fun init(processingEnv: ProcessingEnvironment) {
        super.init(processingEnv)
        AptContext.init(processingEnv)
    }

    //生成代码
    override fun process(annotations: MutableSet<out TypeElement>, roundEnv: RoundEnvironment): Boolean {
        //拿到注解标注的类
        roundEnv.getElementsAnnotatedWith(ModelMap::class.java)
            .forEach {
                    element ->
                //找到构造器
                element.enclosedElements.filterIsInstance<ExecutableElement>()
                        //找到指定的init构造器
                    .firstOrNull { it.simpleName() == "<init>" }
                     ?.let {
                         //转成TypeElement
                        val typeElement = element as TypeElement
                         //传入包名，类名
                        FileSpec.builder(typeElement.packageName(), "${typeElement.simpleName()}\$\$ModelMap")
                            //添加扩展函数
                            .addFunction(
                                //函数名
                                FunSpec.builder("toMap")
                                    //传入Receiver
                                    .receiver(typeElement.asType().asKotlinTypeName())
                                    //传入输出语句
                                    //使用 joinToString 创建语句
                                    .addStatement("return mapOf(${it.parameters.joinToString {""""${it.simpleName()}" to ${it.simpleName()}""" }})")
                                    .build()
                            )

                            .addFunction(
                                FunSpec.builder("to${typeElement.simpleName()}")
                                        //添加泛型参数
                                    .addTypeVariable(TypeVariableName("V"))
                                    .receiver(MAP.parameterizedBy(STRING, TypeVariableName("V")))
                                    .addStatement(
                                        "return ${typeElement.simpleName()}(${it.parameters.joinToString{ """this["${it.simpleName()}"] as %T """ } })",
                                        *it.parameters.map { it.asType().asKotlinTypeName() }.toTypedArray()
                                    )
                                    .build()
                            )
                            .build().writeToFile()
                    }
            }
        return true
    }
}
```



#### 创建文件依赖

**main文件下创建 resources/META-INF/services/ 文件夹****

**并创建 javax.annotation.processing.Processor 文件**,ModelMapProcessor为你的编译类

```
com.android.compiler.ModelMapProcessor
```

![image-20200108140333226](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vm7qwnj30p10am74q.jpg)



#### 如何使用

**appmodel中依赖：**

> 如果在kotlin中使用，依赖注解处理器时必须使用 kapt，而不能使用java中的annotationProcessor.原因是kotlin需要导入相应kapt后，使用时自动去调用java的注解处理器

```kotlin
apply plugin: 'kotlin-kapt'
...

implementation project(':annotations')
kapt project(':compiler')
// 不可使用annotationProcessor
// annotationProcessor project(':compiler')
```



```
package com.cloudx.libnavcompiler

import com.alibaba.fastjson.JSON
import com.alibaba.fastjson.JSONObject
import com.cloudx.libnavannotation.ActivityDestination
import com.cloudx.libnavannotation.FragmentDestination
import com.google.auto.service.AutoService
import java.io.File
import java.io.FileOutputStream
import java.io.IOException
import java.io.OutputStreamWriter
import javax.annotation.processing.*
import javax.lang.model.SourceVersion
import javax.lang.model.element.Element
import javax.lang.model.element.TypeElement
import javax.tools.Diagnostic
import javax.tools.StandardLocation
import kotlin.math.abs

/**
 * Created by Petterp
 * on 2020-02-02
 * Function:
 */
@AutoService(Processor::class)
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@SupportedAnnotationTypes(
    "com.cloudx.libnavannotation.FragmentDestination",
    "com.cloudx.libnavannotation.ActivityDestination"
)
class NavProcessor : AbstractProcessor() {

    private val messager: Messager = processingEnv.messager
    private val filer: Filer = processingEnv.filer
    private lateinit var fos: FileOutputStream
    private lateinit var writer: OutputStreamWriter

    companion object {
        private const val OUTPUT_FILE_NAME = "destnation.json"
    }

    override fun process(
        annotations: MutableSet<out TypeElement>?,
        roundEnv: RoundEnvironment?
    ): Boolean {
        roundEnv?.let {
            val fragmentDestination =
                roundEnv.getElementsAnnotatedWith(FragmentDestination::class.java)
            val activityDestination =
                roundEnv.getElementsAnnotatedWith(ActivityDestination::class.java)

            val destMap = HashMap<String, JSONObject>()
            handleDestination(fragmentDestination, FragmentDestination::class.java, destMap)
            handleDestination(activityDestination, ActivityDestination::class.java, destMap)

            //app/src/main.assets
            val createResource =
                filer.createResource(StandardLocation.CLASS_OUTPUT, "", OUTPUT_FILE_NAME)
            val resourcePath = createResource.toUri().path
            messager.printMessage(Diagnostic.Kind.NOTE, "resourcePath:$resourcePath")

            val appPath = resourcePath.substring(0, resourcePath.indexOf("app") + 4)
            val assetsPath = appPath + "src/main/assets/"

            try {
                val file = File(assetsPath)
                //如果不存在则创建
                if (!file.exists()) {
                    file.mkdirs()
                }

                val outPutFile = File(file, OUTPUT_FILE_NAME)
                if (outPutFile.exists()) {
                    outPutFile.delete()
                }
                outPutFile.createNewFile()

                //
                val content = JSON.toJSONString(destMap)
                //kt自动资源管理
                fos = FileOutputStream(outPutFile)
                fos.use { fileOutputStream ->
                    writer = OutputStreamWriter(fileOutputStream, "UTF-8")
                    writer.use {
                        it.write(content)
                        it.flush()
                    }
                }
            } catch (e: IOException) {
                e.printStackTrace()
            }
        }
        return true
    }

    private fun handleDestination(
        elements: Set<Element>,
        annotationClass: Class<out Annotation>,
        destMap: HashMap<String, JSONObject>
    ) {

        elements.map {
            it as TypeElement
        }.forEach {
            var pageUrl: String? = null
            val id = abs(destMap.hashCode())
            val cazName = it.qualifiedName.toString()
            var needLogin = false
            var asStarter = false
            //区分是Fragment还是Activity
            var isFragment = true
            when (val annotation = it.getAnnotation(annotationClass)) {
                is FragmentDestination -> {
                    pageUrl = annotation.pageUrl
                    asStarter = annotation.asStarter
                    needLogin = annotation.needLogin
                }
                is ActivityDestination -> {
                    pageUrl = annotation.pageUrl
                    asStarter = annotation.asStarter
                    needLogin = annotation.needLogin
                    isFragment = false
                }
            }
            if (destMap.containsKey(pageUrl)) {
                messager.printMessage(Diagnostic.Kind.ERROR, "不同的页面不允许使用相同的pageUrl$cazName")
            } else {
                val objects = JSONObject()
                objects["id"] = id
                objects["needLogin"] = needLogin
                objects["asStarter"] = asStarter
                objects["pageUrl"] = pageUrl
                objects["isFragment"] = isFragment
                destMap[pageUrl.toString()] = objects
            }
        }
    }


}
```



#### 测试类

```kotlin
fun main() {
    val sample = Sample(0, "1","asda")
    val sample2 = sample.toMap().toSample2()
    println(sample)
    println(sample2)
}

@ModelMap
data class Sample(val a: Int, val b: String,val cc:String)

//fun Sample.toMap() = mapOf("a" to a, "b" to b)
//fun <V> Map<String, V>.toSample() = Sample(this["a"] as Int, this["b"] as String)

@ModelMap
data class Sample2(val a: Int,  val cc: String)
```



#### 查看效果

![image-20200108141032979](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vpfhc2j31290dvzln.jpg)



#  Koltin-反射

## 反射的基本

- 反射是允许程序在运行时访问程序结构的一类特性
- 程序结构包括：类，接口，方法，属性等语法特性

![image-20200106105557716](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vqxk3vj30ho0bpt9l.jpg)

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



## 

# Kotlin-协程<*>

## 协程-入门

要点：**挂起，恢复**

### 协程是什么？

- 由程序自行控制 **挂起**，**恢复** 的程序
- 协程可以实现多任务的协作执行
- 协程可以用来解决异步任务控制流的灵活转移



### 协程的作用

- 协程可以让异步代码同步化
- 协程可以降低异步程序的设计复杂度
- 挂起和回复可以控制执行流程的转移
- 异步逻辑可以用同步代码的形式写出

***协程并不会提升性能,协程只是会提高代码的可读性***

### 协程与线程

协程 **Coroutine** : 指语言实现的协程，运行在内核线程上

线程**Thread**: 指操作系统的线程，也称为内核线程



### 关于协程你应该了解的：

- 它是Kotlin标准库
- 协程上下文
- 拦截器
- 挂起函数
- 官方协程框架
- job
- 调度器
- 作用域



## 协程的常见实现

### 协程的分类

#### 按调用栈

- 有栈协程：每个协程会分配单独的调用栈，类似线程的调用栈
- 无栈协程：不会分配单独的调用栈，挂起点状态通过闭包或对象保存(kotlin)

#### 按调用关系

- 对称协程：调度权可以转移给任意协程，协程之间是对等关系
- 非对称协程：调度权只能转移给调用自己的协程，协程存在父子关系

 



## 挂起函数 

- 以 **suspend** 修饰的函数
- 挂起函数只能在 **其他挂起函数** 或 **协程** 中调用
- 挂起函数调用时包含了协程 “挂起” 的语义
- 挂起函数返回时则包含了协程 ”恢复“ 的语义

***Kotlin内部通过 Continuuation 实现挂起函数***

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}

//成功
public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

//异常
public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))
```



### Continuation

***我们一直在说Continuation,那它到底在那里用了呢？*** 用一个例子来证明

#### 挂起函数 -案例

![image-20200108205803900](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vn8c7fj30jt02da9x.jpg)

##### 转成字节码:

![image-20200108205644488](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8viygi4j30kl044gls.jpg)

##### 调用处

> 当转成java字节码后，我们发现它需要 **Continuuation**这个参数。而Continuuation是编译器自动给我们传入的，只存在于挂起函数或者协程中。于是当我们在别的地方调用时，as就会提示报错。

**如果我们不在指定位置调用，则提示如下：**

![image-20200108205715032](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vra9y9j30j103hq2y.jpg)	



#### 将回调转写成挂起函数

##### 将Retrofit请求回调转为挂起函数

```kotlin
interface IBaidu {
    @GET("https://www.baidu.com/")
    fun getBaidu(): Call<String>
}
...
   suspend fun getBaidu() = suspendCoroutine<String> { continuation ->
        Re.retrofit.create(IBaidu::class.java)
            .getBaidu()
            .enqueue(object : Callback<String> {
                override fun onFailure(call: Call<String>, t: Throwable) {
                    continuation.resumeWithException(t)
                }

                override fun onResponse(call: Call<String>, response: Response<String>) {
                    response.takeIf {
                        it.isSuccessful
                    }?.body()?.let(continuation::resume)
                        ?: continuation.resumeWithException(HttpException(response))

                }
            })
    }
```

在上面的🌰中，我们知道，在调用网络请求方法 ***getBaidu***   时，系统已经为我们传入了 ***continuation***，那么应该如何使用呢？

> 我们通过调用 ***suspendCoroutine*** 方法来获得我们的 **Continuation** 接口，从而调用 ***resumeWithException***  实现函数挂起。

而一个函数的真正挂起需要实现异步的调用，在我们上面的🌰中，***enqueue*** 内部已经实现了 io 线程的切换，所以我们可以直接使用 **resumeWithException** 实现函数挂起.

所以 ***真正的挂起*** 必须异步调用 resume,包括以下：

1. 切换到其他线程 resume
2. 单线程事件循环异步执行

相反，如果我们直接 return 或者没有切线程则没有真正挂起。



##### 将回调转写为挂起函数：

- 使用 suspendCoroutine 获取挂起函数的 Continuation
- 回调成功的分支使用 Continuation.resume(value)
- 回调失败则使用 Continuation.resumeWithException(e)





### 协程的创建

- 协程是一段可执行的程序
- 协程的创建通常需要一个函数
- 协程的创建同时需要一个API

 下面我们看一下创建协程与启动的源码：

```kotlin
//创建协程
public fun <R, T> (suspend R.() -> T).createCoroutine(receiver: R,completion: Continuation<T>): Continuation<Unit> =
    SafeContinuation(createCoroutineUnintercepted(receiver, completion).intercepted(), COROUTINE_SUSPENDED)

//启动协程
public fun <T> (suspend () -> T).startCoroutine(
    completion: Continuation<T>
) {
    createCoroutineUnintercepted(completion).intercepted().resume(Unit)
}
```

从上面我们可以看出，创建一个协程需要传入一个 ***Continuation***，但同样它也返回了一个 ***Continuation***（为我们协程创建出的本体）。

总结如下：

- ***suspend*** 函数本身执行需要一个 ***Continuation*** 实例在恢复时调用，即此处的参数：***completion***
- 返回值 **Contination<Unit>** 则是创建出来的协程的载体，receiver **suppend** 函数会被传给该实例作为协程的实际执行体。



### 协程上下文(CoroutineContext)

```kotlin
   suspend {  }.startCoroutine(object:Continuation<Unit>{
       override val context: CoroutineContext
           get() = TODO("not implemented") //To change initializer of created properties use File | Settings | File Templates.

       override fun resumeWith(result: Result<Unit>) {
           TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
       }
   })
```

![image-20200108212822271](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vjdp48j30kx075js8.jpg)

- 协程执行过程中需要携带数据
- 索引是 CoroutineContext.key
- 元素是 CoroutineContext.Element



### 拦截器

- 拦截器 ***ContinuationInterceptor*** 是 协程上下文(CoroutineContext) 元素
- 可以对协程上下文所在协程的 ***Contiunation*** 进行拦截

![image-20200108213546179](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt8vijfygj30qc0afwgi.jpg)



#### Continuuation执行流程图



  协程挂起恢复要点

- 协程体内的代码都是通过 Continuation.resuumeWith 调调用
- 每调用一次 label 加1，每一个挂起点对应一个 case 分支
- 挂起函数在返回 CORUTINE_SUUSPENDED 时才会挂起



## 协程的启动模式

| 启动模式     | 功能特性                                                     |
| ------------ | ------------------------------------------------------------ |
| DEFAULT      | 立即开始调度协程体，调度前若取消则直接取消                   |
| ATOMIC       | 立即开始调度，且在第一个挂起点不能取消                       |
| LAZY         | 只有在需要(start/join/await)时开始调度                       |
| UNDISPATCHED | 立即在当前线程执行协程体，直到遇到第一个挂起点(后面取决于调度器) |



#### 其他特性

- Channel 热数据流，并发安全的通信机制
- Flow :冷数据流，协程的响应式 API
- Select 可以对多个挂起事件进行等待



## Channel

- 非阻塞的通信基础设置
- 类似于 BlickingQueue+挂起函数



