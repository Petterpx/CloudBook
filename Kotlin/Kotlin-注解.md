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

  