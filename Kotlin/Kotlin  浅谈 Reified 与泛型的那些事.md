# Kotlin | 浅谈 Reified 与泛型 的三两事



## 背景

在业务中，或者要写某个技术组件时，我们无可避免会经常使用到 `泛型` ，从而让代码更具复用性与健壮性。

但相应的，由于Java泛型存在 **类型擦除** 的实现机制，所以某些情况下就会显得力不从心。而在 `Kotlin` 中，由于最终也会被编译为java字节码，所以无可避免也存在这上述问题🙂。

**什么是类型擦除?**

如下例所示:

```kotlin
Class c1=new ArrayList<Integer>().getClass();
Class c2=new ArrayList<String>().getClass();
// 输出为true
System.out.println(c1==c2);
```

`上述输出结果为什么会true呢？`

因为泛型从底层上说是一种语法糖，它只存在于 **编译期** 。在代码运行期间，jvm会将泛型的相关信息擦除，成功编译后的 **class文件** 不会包含任何泛型信息。所以在运行时，c1与c2存储的类型都为Object,而 Integer 与 String 已经被擦除。

正是由于类型被擦除，所以就导致一些相关问题，比如不安全的类型转换,模版方法的增多等。

比如下面例子所示：

```java
ArrayList list = new ArrayList();
list.add(1);
list.add("@");
```

## reified

为了解决上述类型擦除问题，以及更好的使用体验。`Kotlin` 中存在名为 `reified` 的关键字,它可以被作用于函数上, 以此做到类型擦除后的再生，便于开发者优雅的使用泛型以及获取方法的泛型类型。

在 `Java` 中，如果我们要获取函数的泛型类型，一般会通过给函数中传递类型参数的方式，如下所示：

```java
public <T extends Activity> void startActivity(Context context, Class<T> c) {
     context.startActivity(new Intent(context, c));
 }
```

如果上述代码使用 `kotlin` 来写呢？如下所示:

```kotlin
inline fun <reified C : Activity> Context.startActivityKtx() {
    startActivity(Intent(this, C::class.java))
}
```

> 我们利用 **扩展函数** + `reified` 关键字的方式，减少了模版代码，增强了使用体验。
>
> 从而让本该在编译阶段被擦除的Activity类型，能够在运行时获取到。

但需要注意的是，`reified` 关键字必须和 `inline` 关键字一起使用(下面会提到为什么)。

**inline 关键字是什么呢？**

> 简单理解为：当一个函数被标记为 `inline` 时，**kotlin编译器** 会在所有调用这个函数的位置，将方法函数替换为具体的函数体。

## 解析

通过查看 kotlin 字节码，我们可以得知 `reified` 的底层实现。

例如下面示例与其对应的字节码:

```kotlin
inline fun <reified C : Activity> Context.toAct() {
    startActivity(Intent(this, C::class.java))
}

fun test(context: Context) {
    context.toAct<MainActivity>()
}
```

```java
// $FF: synthetic method
public static final void toAct(Context $this$toAct) {
   int $i$f$toAct = 0;
   Intrinsics.checkNotNullParameter($this$toAct, "$this$toAct");
   Intrinsics.reifiedOperationMarker(4, "C");
   $this$toAct.startActivity(new Intent($this$toAct, Activity.class));
}

public static final void test(@NotNull Context context) {
   Intrinsics.checkNotNullParameter(context, "context");
   int $i$f$toAct = false;
   context.startActivity(new Intent(context, MainActivity.class));
}
```

我们在 test() 方法中调用toAct(),不难发现,toAct()的逻辑已经被移动到了 test() 中,而我们的泛型类型也被替换为实际使用的类型，从而我们可以在方法函数中直接获取相应的泛型类型。

这也就是为什么 `reified` 必须要增加 `inline`  ,因为其必须内联才能知道具体类型，从而将我们的实际泛型类型更新到具体的调用代码中,从而完成泛型类型再生。


## 小提示

### Java中无法调用

需要注意的是,`reified` 无法在java中进行调用，为什么呢？

因为 `Java` 并没有内联的特性，我们使用的 `inline` 方法在 `Java` 中会被当做普通方法，而 `reified` 正是需要内联才可以保证泛型再生，所以自然无法调用。

从源码上来说，对于reified 关键字的方法,相应的字节码生成时会增加 `// $FF: synthetic method` 的标记。

如下示例所示：

```kotlin
inline fun <reified C : Activity> Context.toAct() {
   ...
}
```

```kotlin
// $FF: synthetic method
public static final void toAct(Context $this$toAct) {
   ...
}
```

**// $FF: synthetic method**

如果你经常写组件，肯定见过这段注释。你可以理解这只是一个标记，其作用为告诉编译器 **禁止java代码在编译期访问该方法** 。

比如我们在写 kotlin 组件，而且要同时满足 java 调用时,经常会免不了使用 `internal` ,即 **模块可见** 。但相应的，该关键字修饰的方法或者字段在Java中却依然可以被调用，甚是让java调用者费解与不优雅。所以相应的，对于方法，我们可以增加  `@JvmSynthetic` ,从而避免java代码编译期调用。而 `reified` 也是正是采用了该思路。

> 当然也可以采用 `@JvmName(name=" xxx")` 等方式避免java调用,但其并不是很优雅。

### 性能方面

使用 `reified` 不会带来任何性能损失，相反还会增强性能(源于`inline`)。

之所以这里还要提到，主要是因为如果 内联函数过于复杂，则可能会导致性能问题。所以 reified 的使用其实也需要遵循内联函数的最佳实践。

如果查看Kotlin的标准内联函数,你会发现，代码行数大部分只有1-3行，因为inline会增加代码量的生成，内联函数越复杂，相应的代码量也越高,具体的使用方面，可以参见这篇 [Kotlin Vocabulary | 内联函数的原理与应用](https://zhuanlan.zhihu.com/p/149927629)。



## 参考

[Kotlin Vocabulary | Reified: 类型擦除后再生计划](https://segmentfault.com/a/1190000024518568)
