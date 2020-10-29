# Flutter入门-widget

这一章相比来说比较繁琐，主要都是一些概念需要掌握。



## 概念

在以往原生的开发中，我们有各种UI控件,比如Android中的TextView等等,但 Flutter中的Widget不同于原生中的控件，它不仅可以表示UI元素，也可以表示一些功能性的组件：比如用于手势监测的 **GestureDetector** widget,用于app 主题数据传递的 Theme 等，但由于Flutter主要用于构建用户界面，所以我们可以直接认为 widget 就是一个控件。

## Widget和Element

在Flutter中，**Widget 仅仅是一个描述显示元素的配置数据(官方解释)**，而真正代表屏幕上显示元素的是 **Element(相当于一个纽带，用于连接widget和具体渲染的一个中间人)** ,所以可以理解为，**widget只是ui元素的一个配置数据，并且一个widget可以对应多个Element.**这是因为在实际渲染时，UI 🌲 上的每一个 **Element** 节点都会对应一个Widget对象。

> 上面这个描述可能听起来有些绕口，但是暂时你可以直接认为，widget不是实际屏幕显示元素，它仅仅只是描述了要显示的实际元素的属性，然后在实际运行中，flutter 会将每一个widget与每一个element对应。

- Widget 实际上就是 Element 的配置数据，Widget树实际上是一个配置树，而真正的渲染树是由 Element 构成，不过，由于Element 是通过Widget生成，它们之间存在对应关系，所以大多数场景下，我们可以大体上认为 Widget树就是指 UI控件树或者UI渲染树。
- 一个Widget对象可以对应多个 **Element** 对象，可以理解为，同一份配置(widget) 可以创建多个实例 （Element）

## Widget主要接口

```dart
@immutable
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key key;

  @protected
  Element createElement();

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

- **widget** 继承自 **DianosticableTree** ，**DiagnosticableTree** 即诊断书，主要作用是提供调试信息。
- **key:** 这个 **key** 属性类似于React/Vue 的 **key**,主要的作用是决定是否在下次 **build** 时复用旧的widget ,决定的条件在 **canUpdate()** 方法中。
- **createElement()**  Flutter Framework在构建UI树时，会先调用此方法生成对应节点的 **Element** 对象，此方法是 Flutter Framework 隐私调用的。
- **debugFillProgerties(..)** 复用父类的方法，主要是设置诊断树的一些特性。
- **canUpdate(..)** 是一个静态方法，它主要用于在Widget 树 重新 **build** 时复用旧的 widget。



## Context

**build** 方法有一个 context参数，它是 **BuildContext** 类的一个实例，**表示当前 widget 在 widget中的上下文**，每一个 widget 都会对应一个 context 对象。实际上， **context** 是当前widget在widget树中任意位置中执行相关操作的一个句柄。

比如它提供了从当前 widget 开始向上遍历 widget树以及按照 widget类型 查找父级widget的方法 **findAncestorWidgetOfExactType**。

```dart
context.findAncestorWidgetOfExactType<xxWidget>();
```



## StatelessWidget 和 StatefulWidget

Flutter 中的 Widget包含两种，一种是不需要更改状态的 Widget,也就是 **StatelessWidget**,另一种是可变状态的 **StatefulWidget**,注意这里所说的状态都是Widget里的状态 **State**，而管理状态一般是通过 **setState** 来管理。

### 那什么是有状态和无状态呢？

- **有状态：** 交互或者数据改变导致 Widget改变，例如改变文字
- **无状态：**不会被改变的 Widget,比如一个纯页面的展示

> **需要注意的是，使用 StatefulWidget 时，每次直接setState会导致整个widget全部重建，所以在使用时，我们应该尽量把 子widget 抽离出去，采用局部刷新的方式优化，当然这个技巧具体可以百度或者参阅我之前的代码，并不是什么高难度骚操作。**



## State

一个 StatefulWidget 类会对应一个 State类，State表示与其对应的 statefulWidget 要维护的状态，State中的保护的状态信息可以：

1. 在widget构建时可以被同步读取；
2. 在widget生命周期改变时可以被读取，当 State 被改变时，可以手动调用 其 **setState()** 方法通知 Flutter framework状态发送改变，Flutter framework在收到消息后，会重新调用其 **build** 方法重新构建 widget 树，从而达到更新UI的目的.

State 中有两个常用属性：

1. **widget:** 它表示与该State 示例关联的 widget 实例，由 Flutter framework 动态设置，不过这种关联并非永久，因为在应用生命周期中，UI树上的某一个节点 widget 示例在重新构建时可能会变化，但 State 实例只会在第一次插入到树中时被创建，当在重新构建时，如果 widget 被修改了，Flutter framework 会动态设置State, widget为新的 widget 实例。
2. **context** StatefulWidget 对应的 BuildContext, 作用同 StatelessWidget 的 BuildContext一致。



### State生命周期

#### **initState**

当 Widget 第一次插入到 Widget树时被调用。对于每一个 State 对象，Flutter framework只会调用一次回调。适合做一些一次性的操作，比如状态初始化，订阅子树的事件通知等。

不能在 该回调 中调用 **BuildContext.dependOnInheritedWidgetOfExactType**,原因是在初始化完成后， Widget 树中的 **InheritFromWidget** 也可能会发生变化，所以正确的做法应该在 **build** 方法或 **didChangDependencied()** 中调用它。

#### didChangeDependencies()

当State 对象的依赖发生变化时被调用。

#### build

主要用于构建 Widget 子树时被调用，它会在如下场景被调用：

1. 在调用 **initState** 之后
2. 调用 **didUpdateWidget** 之后
3. 调用 **setState** 之后
4. 调用 **didChangeDependencies** 之后
5. 在 State 对象从树中一个位置移除后，又重新插入到树的其它位置之后

#### reassemble

此回调是专门为开发调试而提供，在热重载 (hot reload) 时被调用，此回调在 release 下永远不会被调用。

#### didUpdateWidget

widget重建时，如果新旧 widget 的key相同就会调用此方法

#### deactivate()

当State对象从树中被移除时，会调用此方法。在一些场景下，Flutter framework 会将State 对象重新插入到树中，如包含此 State 对象的子树在树的一个位置移动到了另一个位置时。如果移除后没有重新插入到树中则紧挨着会调用 **disponse** 方法。

#### disponse()

当State对象从树中被永久移除时调用，通常用于在此回调中释放资源。

#### 生命周期图

![image-20201029085515826](https://tva1.sinaimg.cn/large/0081Kckwly1gk5x6nerlwj30kn0jbtag.jpg)

[Process保存地址](https://www.processon.com/view/link/5f9a0bd85653bb4bebc441d4)

#### 具体动画示例

<figure class="third">     <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk5xjfetwog30by0mwqv9.gif" width="300"/><img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk5xmrjusgj30zs0e0tcs.jpg" width="600"/>
> 左边是动画，右边对应着日志变化。~
>



