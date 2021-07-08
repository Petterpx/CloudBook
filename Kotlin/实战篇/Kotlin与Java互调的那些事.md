[]: 

# Kotlin与Java互调的那些事

![image-20201015161230476](https://tva1.sinaimg.cn/large/007S8ZIlly1gjq35a8gpnj312c0hmn1k.jpg)

### Kt调用-Java参数非null的处理

#### @NotNull

**Java**

```java
class TestJava {
    public void toNotNull(@NotNull String title) {}

    public void toNull(String title){}
}
```

**Kotlin中调用**

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gjptlg2kzvj30lo04yjrw.jpg" alt="image-20201015104201440" style="zoom:150%;" />



### Kt调用- Java中使用kt关键字声明的变量和方法

 kotlin中调用java方法，参数时，如果含有Kotlin关键字，必须增加 反引号 ``

**Java**

```java
public Object object;
//使用kotlin中关键字命名的方法
public void is() {

}
```

**Kotlin中调用**

```kotlin
 testJava.`is`()
 testJava.`object`
```



### Kt调用Java-SAM转换

在Kotlin中调用带有接口参数的方法时，如果这个接口只有一个方法，那么就可以通过 lambda 表达式实现 SAM转换，从而简化我们的代码。

**示例如下**

**Java**

```java
public class TestJavaSam {
  
    void singleFun(@NotNull IListener iListener) {}

    void noParameter( @NotNull IListener iListener,int sum) {}

    void noParameterClean(int sum,@NotNull IListener iListener) {}
}

interface IListener {
    void onClick();
}
```

**在Kotlin中调用**

```kotlin
fun main() {
    val sam = TestJavaSam()
    sam.singleFun {

    }
    sam.noParameter({
			//如果更改一下java方法参数的顺序，那么就会更简洁，如下
    },123)
    sam.noParameterClean(123){

    }
}
```

​	

### Kt中禁止Java调用某方法

#### @JvmSynthetic

在Kotlin中，有些方法并不想暴露给Java调用，这时就可以增加这个注解在方法上。

```kotlin
@JvmSynthetic
fun toMain() {

}
```

此时toMain() 在Java中将无法调用到。





### Java调用Kt-扩展函数

#### @file:JvmName("xx")

在java中使用Kotlin的扩展函数时，我们都会使用相应的类名+Kt 去调用相关的方法，有时候我们想自定义相应的工具类，就显得稍显麻烦，如下：

比如我们有一个顶级扩展函数，位于 **UiExpand.kt** 中：

```kotlin
fun Int.px() {}
```

**Java中调用 **

```java
  //Java调用kotlin类-(UiExpand)-Int.px() 扩展方法
  UiExpandKt.px(20);
```

如上所示，在Java中调用时，我们必须已文件名+kt后缀才可以调用。

通过给 UiExpand.kt 包名上增加 **@file:JvmName("Ui")**，我们就可以实现自定义生成的类名去调用

```java
如下
Ui.px()
```







### Java调用kt-成员变量

#### @JvmField

在Java中，我们去调用Kotlin 的 成员变量 时，编译器都会帮我们自动生成相应的 get,set方法，这很符合Java Bean的写法，但是有些是有我们只是想直接去调用，这个时候就可以这样去做。

**Kotlin**

```kotlin
data class TestKotlinBean(
  @JvmField val message: String, 
  @JvmField val title: String)
```

**Java中调用 **

```java
 TestKotlinBean testKotlinBean = new TestKotlinBean("","");
 String message = testKotlinBean.message;
 String title = testKotlinBean.title;
```

当然对于 如下的示例，就算不用增加上面的注解，在java也都是可以直接调用，免除get,set。

```kotlin
lateinit var sum: String

object UserPicCache{
        const val KEY_CACHE = "CACHE"
 }
```



#### @set:JvmName

#### @get:JvmName

有些时候，我们只是想让其生成其中的一个set或者get方法，这个时候应该怎么做呢？

**Kotlin**

```kotlin
data class TestKotlinBean(
    @set:JvmName("setMessage")
    var message: String,
    @get:JvmName("getTitle")
    val title: String
)
```

**Java中调用**

```java
 TestKotlinBean testKotlinBean = new TestKotlinBean("", "");
 testKotlinBean.setMessage("message");
 testKotlinBean.getTitle();
```





### Java调用Kt-伴生对象

#### @JvmStatic

当我们在Java中调用 Kotlin 伴生对象的方法或者变量时，必须通过  **类名.Companion.xx** 的方式才可以调用。这时候，我们就可以增加

@JvmStatic 来直接调用。

**Kotlin**

```kotlin
class Log {
    companion object {
        var time: String = ""
        fun toLog() {
        }
    }
}
```

**在Java中调用**

```java
 ToLog.toLog();
 ToLog.getTime();
```

不过需要注意的是，**@JvmStatic** 对性能没有任何提升，因为相应的，编译器又生成了一个静态方法，对于可变变量，会生成两个静态方法set,get。

![image-20201015150453429](https://tva1.sinaimg.cn/large/007S8ZIlly1gjq16zsudnj30m90l7ju2.jpg)



### Java调用Kt-方法默认参数值

#### @JvmOverloads

在Kotlin中，对于方法参数，我们可能会加入一些默认值，便于更好的使用，但是在Java中，如果调用时不传递相应的方法参数，就会提示报错，这种使用就可以使用 ***@JvmOverloads*** 修饰方法。

**kotlin**

```kotlin
object DialogUtils {
    @JvmStatic
    @JvmOverloads
    fun showPromptDialog(title: String = "提示") {
    }
}
```

**在java中调用**

```java
DialogUtils.showPromptDialog();
DialogUtils.showPromptDialog("标题");
```





参阅资料：

[Google开发者-如何在 Java 和 Kotlin 之间进行互操作](https://mp.weixin.qq.com/s?__biz=MzAwODY4OTk2Mg==&mid=2652051518&idx=3&sn=78f0e5afb8eb42b86166ab9dbeef46ae&chksm=808cb87bb7fb316d0b54564a4543de13151f44cb9dfdafdc75aac6038f4a9a24ce91e82ce668&scene=178#rd)

