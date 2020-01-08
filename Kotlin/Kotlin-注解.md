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

![image-20200108140333226](https://tva1.sinaimg.cn/large/006tNbRwly1gap4ak765cj30p10ammxw.jpg)



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

![image-20200108141032979](https://tva1.sinaimg.cn/large/006tNbRwly1gap4hsfqfij31290dvtau.jpg)







