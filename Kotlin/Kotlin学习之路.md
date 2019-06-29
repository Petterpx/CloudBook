# Kotlin学习之路

### 什么是Kotlin? Kotlin学习指南

```
Kotlin就是一门可以运行在 jAVA虚拟机，Android,浏览器上的静态语言，它与Java100%兼容，如果你对Java非常熟悉，那么你就会发现Kotlin除了自己的标题库之外，大多任然使用经典的Java 集合框架。

学习目标
1. 学会使用 Kotlin
2. 熟悉Java生态
3. 了解一些特性背后的实现

```

---



### 一些快捷键

main  main方法快捷

sout println快捷





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

![1549726301870](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549726301870.png)

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
   - ![1549882894244](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549882894244.png)

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

![1549897091828](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549897091828.png)

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

我们看一下字节码的变化

![1549966140172](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549966140172.png)



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

![1549966062447](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549966062447.png)

所以这就是编译器常量的作用。编译器常量的引用都是直接指明该变量的值



---

#### 函数

![1549972925834](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1549972925834.png)

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
      val list = listOf(
          1..20,
          6..100,
          200..220
      )
      //flatMap相当于 把几个小的list转换到一个大的List
  //    list.flatMap { it }.forEach(::println)
  
      //如果现在要给每个值前加上 No.
      list.flatMap { intRange ->
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


