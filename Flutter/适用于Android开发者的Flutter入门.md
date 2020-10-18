# 适用于Android开发者的Flutter入门



## View

### Flutter中的widget和Android的View有什么不同？

相比Android中的View,Widget 仅支持一帧，并且在每一帧上，Flutter的框架都会创建一个 widget 实例树。(也就是相当于一次性绘制整个界面)，相比之下，在Android 上 View 绘制结束后，就不会重绘，知道调用 **invalidate** 时才会重绘。

### 如何更新 widget?，什么是有状态及无状态widget ?

在Android中，可以通过直接对View进行改变来更新视图，然而，在Flutter中，Widget 是不可变的，并不会直接更新，而必须使用 Widget 的状态。

这便是 **StateFul** 和 **Stateless Widget**的概念来源。

- **Stateless Widget** 没有状态信息的 widget ， 当需要的用户界面不依赖与对象配置信息以外的其他任何内容时使用。
- **Stateful Widget** 。 如果视图需要随着相应的状态更改，则必须使用 StatefulWidget ,并告诉Flutter 框架 该 Widget 的状态已经更新。

**注意：** 无状态和有状态 widget 的核心特性是相同的，每一帧他们都会重新创建，不同之处在于 **StatefuleWidget** 有一个 State 对象，它可以跨帧存储状态数据并恢复它。

上述的解释，通俗的理解为：

**如果一个 widget 发生变化(例如用户点击)，它就是有状态的，但是，如果一个子 widget 对变化做出反应，而其父 widget 对变化没有任何反应，那么包含的 父 widget 仍然是无状态的 widget**

示例如下：

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(SampleApp());
}

class SampleApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: "petterp",
      theme: ThemeData(
          primarySwatch: Colors.blueGrey,
          visualDensity: VisualDensity.adaptivePlatformDensity),
      home: SampleAppPage(),
    );
  }
}

class SampleAppPage extends StatefulWidget {
  @override
  _SampleAppPageState createState() => _SampleAppPageState();
}

class _SampleAppPageState extends State<SampleAppPage> {
  String num = "45";

  void _updateNum() {
    setState(() {
      num = "123";
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(num),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _updateNum,
        tooltip: "update Text",
        child: Icon(Icons.add),
      ),
    );
  }
}

```



### 如何在布局中添加或删除组件？

在Flutter中，因为 widget 是不可变的，所有没有 addChild,相反，可以通过传入一个 函数，该函数返回一个 widget 给父项，并通过布尔值控制 该 widget 的创建。

示例

```dart
class _SampleAppPageState extends State<SampleAppPage> {
  var toggle = true;

  void _updateChild() {
    setState(() {
      toggle = !toggle;
    });
  }

  _getChild() {
    if (toggle) return Text("Toggle One");
     return Text("Toggle Two");
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("remove widget"),
      ),
      body: Center(
        child:_getChild(),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _updateChild,
        tooltip: "update Text",
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### 

### Flutter中对动画进行处理



- 
- [test]()

## Intent

### 在Flutter中处理来自外部应用程序传入的Intent

Flutter 可以通过直接与Android 层通信并请求共享的数据来处理来自 Android 的 Intent。





