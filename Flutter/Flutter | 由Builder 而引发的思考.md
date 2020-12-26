

![image-20201026131503372](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65b9cc889930469e8ede35dbdcff2a4d~tplv-k3u1fbpfcp-zoom-1.image)

## 概要

本篇主要是我实际学习中遇到的一个问题，从而引发的一些思考，从本篇你将学到如下：

- **Builder** 神奇却又简单的背后缘由
- **BuildContext** 的真实理解
- **widget** 与 **element** 的关系，及流程分析

## 背景

关于 **Builder** 这个widget,我想大家都是通过报错才发现的有这个widget的。

比如 *From.of(context)* ，为什么null指针(Dart新特性)了，*Navigator.maybePop(context)* 怎么异常了，诸如此类需要 `context` 传入的地方。

于是我举一个最简单的例子如下：

|                             代码                             |                             图示                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201221002737528](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfd3811847be4c75981a1410a10b1381~tplv-k3u1fbpfcp-zoom-1.image) | ![Dec-21-2020 00-26-38](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d05c5fb8ed442e7aee324cb21abf280~tplv-k3u1fbpfcp-zoom-1.image) |

> 详细解释如图所示。使用Form来验证我们的输入框是否输入合格。

作为一个Flutter新手，肯定会好奇，我为啥null了呢，然后google一搜，就有人建议你使用 Builder,然后我们就会将代码改为以下方式：

|                             代码                             |                             图例                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201221003757405](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2165dbf42a54e51a61d6bf2f82e2964~tplv-k3u1fbpfcp-zoom-1.image) | ![Dec-21-2020 00-32-07](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bef573e2530042c1b01a6d8a39ee8f74~tplv-k3u1fbpfcp-zoom-1.image) |

欧耶，好啦，就这么简单啊。

等等，好像有什么地方不对，作为一个由良知，有道德，遵纪守法，爱国，不掉头发的 新时代无产阶级 干饭人，疯狂套用似乎不符合我的气质，我决定深入细节，看看你这葫芦里卖的什么药。

---

## Builder 是什么？

官方解释：

> 一个无状态实用程序小部件，其[build]方法使用其[builder]回调创建小部件的子级。
>

**源码如下**

```dart
class Builder extends StatelessWidget {

  const Builder({
    Key? key,
    required this.builder,
  }) : assert(builder != null),
       super(key: key);
       
  @override
  Widget build(BuildContext context) => builder(context);
}
```

当我看过Builder的源码，我从没感觉自己如此机智且聪明，是个开发者都能看懂。很简单，就尼玛的一个接口回调，这是不是随手都能写一个出来。

---

## 缘由

**那为什么我自己的context不行呢？**

让我们先去看看 **Form.of** 方法，当然其他of的方法也类似。

```dart
static FormState? of(BuildContext context) {
  //获取给定类型为T的最近的小部件，该类型必须是具体的[InheritedWidget]子类的类型，并向该小部件注册该构建上下文，以便在该小部件发生更改时（或引入该类型的新小部件时，或窗口小部件消失），将重新构建此构建上下文，以便它可以从该窗口小部件获取新值
  final _FormScope? scope = context.dependOnInheritedWidgetOfExactType<_FormScope>();
  return scope?._formState;
}
```

咳咳，简单理解 **dependOnInheritedWidgetOfExactType**  

这个方法会根据我们传递进去的context，去从它的父级开始向上查询与当前 给定的类型匹配以及最近的这个widget,如果找到就返回，否则就抛异常。

然后 让我们将视角切换到最开始的截图，注意我圈出来的地方。

![image-20201221004613004](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3894f329167e463a99c4e69acb8d3405~tplv-k3u1fbpfcp-zoom-1.image)

哦，我可真是个小菜瓜啊，我传递个根context进去，你让From.of() 怎么找，它的父Widget树向上怎么可能有FromState。

---
知道了缘由，甚至于你自己可以不用 Builder,自己写一个小组件也行，比如下面示例。

|                           示例代码                           |                             动画                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201224215000620](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24d1a89f1193415ab80fba0ea55a2c98~tplv-k3u1fbpfcp-zoom-1.image) | ![Dec-24-2020 21-51-19](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d14cdf51a88423585ddba616821add3~tplv-k3u1fbpfcp-zoom-1.image) |

---

## 思考

但是到这里就结束了吗？如果对于本篇而言，的确是。但对我自己而言，却带来了更多疑问：

- **context** 到底是干什么的?
- **build(context)** 方法中的 `BuildContext` 是哪里来的？



### Widget和Element的关系

我们常听说 Flutter有三棵树，也就是 **Widget** , **Element** ，**RenderObject** ，我们主要关心前两者。

**Widget** 树，顾名思义，就是我们常用的组件，其仅仅相当于我们对 `UI` 元素的一个 `配置`。

**Element,是Widget 实际对应的对象**。why?没懂，没错，其实我也没明白😂

> 我们通过源码分析一下，Flutter的源码相比Android原生，是非常简单好理解。

我们以常使用的 **StatelessWidget** 为例来看看。

`StatelessWidget`

```dart
abstract class StatelessWidget extends Widget {
  ...
  @override
  StatelessElement createElement() => StatelessElement(this);
  @protected
  Widget build(BuildContext context);
}
```

我们常见的 **build(context)** 方法，是其定义的一个抽象方法；

其实现了 **Widget** 的抽象方法 **createElement()**，并传入了我们当前实例对象，所以继续往下看。

---

`StatelessElement`

```dart
class StatelessElement extends ComponentElement {
  /// Creates an element that uses the given widget as its configuration.
  StatelessElement(StatelessWidget widget) : super(widget);

  @override
  StatelessWidget get widget => super.widget as StatelessWidget;
	
  //划重点
  @override
  Widget build() => widget.build(this);
}
```

我们主要看 **build()** 方法，其调用的我们 `StatexxWidget`-**build** 方法，其实现widget的构建，并传入了一个this,也就是一个StatelessElement,但是我们最终拿到的都是 BuildContext啊？这是为什么呢，我们继续去看 **ComponentElement**。

```dart
Widget build(BuildContext context)
```

---

`ComponentElement`

```dart
//用于构建窗口小部件
abstract class ComponentElement extends Element 
```

`Element`

```dart
  abstract class Element extends DiagnosticableTree implements BuildContext 
```

原来Element实现了 **BuildContext** 接口，所以在StatelessElement-build()中，可以直接传递element进去。

我们看一下官方对 **Element** 的解释：

![image-20201224230229659](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd23ba0bad09448981dbeeafea8764a2~tplv-k3u1fbpfcp-zoom-1.image)

简而言之，就是，**Element** 代表了 **Widget** 在树中实际位置的实例对象，为什么这么说呢？

因为Widget实际上就是`Element`的配置数据，**Widget** 树也就是一个配置树，而真正的 **UI** 渲染树是由`Element`构成；不过，由于`Element`是通过Widget生成的，所以它们之间有对应关系；相应的，一个Widget对象可以对应多个`Element`对象。这也很好理解，根据同一份配置（Widget），可以创建多个实例（Element）。

`BuildContext`

![image-20201224231620813](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a9a5d52573b4f679c9d9546fa951ad5~tplv-k3u1fbpfcp-zoom-1.image)

我们可以理解为 BuildContext 对象实际就是 Widget对应的 Element对象.所以我们可以通过 context 在StatelessWidget 和 StatefulWidget 的build方法来间接的访问element对象(通过各种xx.of)，而我们开发中 widget的组合使用，比如各种Widget的搭配，由它们形成了我们的配置树，而这个widget最终会一一对应一个 element，从而形成了一个Emelent树。

---

**但为什么build方法里，不直接定义成Element对象，却要定义为BuildContext?**

官方的解释如下：

**BuildContext 用于阻止对 Element 的直接操作。**

显然这个解释并不是怎么好理解，在反复思考及挠头后，我个人的理解如下：

> 在软件开发中，任意复杂耦合的两个事物之间都可以通过一个第三者进行解耦，而BuildContext就是如此。
>
> 因为我们的 Element 承担了widget 实际的对应对象，相应的其有很多初始化及其他方法是不便于我们开发者直接调用的，如果将其直接暴露出去，相应的复杂度会大大提示，所以它通过 BuildContext这个接口，并定义了相应的一些操作 Element 的方法，虽然一定程度上来说，我们依然能间接操作 element,但是通过这种第三者的方式，很好的屏蔽了一些特性，对于我们开发者而言，只需关注widget即可，对于element相关的操作，可以通过相应Widget的xx.of() 方法，极大程度上让我们开发者可以更专注的应该widget层的开发，而无需关注其他方面。

---

### 总结

#### *1.context?*

> context是什么？

我们常用的 **widget** 只是一个配置信息，实际每一个 widget 都对应了一个 `Element `,由于 Element 实现了 `Buildcontext` ,所以在 `StatexxxWidget`-**build(context)** 方法里，通过 `context` ,我们可以间接的操作 Emelent 去进行一些操作，比如 `xx.of(context)` 内部正是调用了 dependOnInheritedWidgetOfExactType (即从Element父级开始寻找匹配的widget)，所以我们可以认为:

context实际就是我们widget在Element树中对应的实际位置。



#### *2.build(context)?*

> **build(context)** 方法中的 `BuildContext` 是哪里来的？

这个问题实际上就是对源码做了一个简单概括：

- 我们常用的 `StatelessWidget` 或者 `StatefulWidget`,其内部 **build()** 或者后者 `State`-**build()** 方法，都是返回一个 `Widget` 对象；
- 而上述的两个组件都继承自 `Widget` ,有一个 **createElement()** 的抽象方法，此方法默认是返回了一个 `StatexxxElement(this)`，其 `this` 代表当前 **widget** 实例;
- 而 StatelessElement 继承自 `ComponentElement` ，其 **build()** 默认是调用了我们 **Widget** 中的 build(context) 方法从而实现了初始化；
- 由于 ComponentElement 继承自 `Element` ，Element 实现了 `BuildContext` 接口,所以我们可以在 Widget 的 build(context) 方法中拿到 BuildContext;


