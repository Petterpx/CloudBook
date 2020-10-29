

# Flutter入门-路由导航





Flutter入门系列连载：

- Flutter入门-路由导航-[本文对应代码链接](https://github.com/Petterpx/Flutter-Example-Demo/blob/master/lib/no1/navigator_demo.dart)

<br/>

### 什么是路由？

> 首先什么是路由，路由对于移动开发者来说就是页面，比如对于我们Android开发者来说就是 Activity A-> ActivityB，类似ios中的 ViewController。

> 而人们常常说起的路由管理，就是管理页面之间如何跳转，通常也可被称为导航管理。

例如：

<img src="//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9c03e84e15f4143bf255cb969e3230c~tplv-k3u1fbpfcp-zoom-1.image" alt="Oct-23-2020 18-09-25" style="zoom:50%;" />

### MaterialPageRoute

MaterialPageRoute 继承自 PageRoute类，是 Material 组件库提供的组件，针对不同平台，其有不同的路由动画效果。

其中PageRoute 是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，其定义了路由构建及切换过渡动画的接口及属性。

```dart
MaterialPageRoute({
    WidgetBuilder builder,
    RouteSettings settings,
    bool maintainState = true,
    bool fullscreenDialog = false,
  })
```

- **builder** 

  用于构建路由页面的具体内容

- **Settings** 

  包含路由的基本配置信息，如名称，是否为初试路由页(首页)

- **maintainState** 

  默认打开一个新页面时，保存当前原路由信息。设置为false时，在入栈新页面时，释放当前原路由所占用的资源

- **fullscreenDialog** 

  新路由是否是一个全屏的模态对话框，例如在ios中，如果为true,则新页面从屏幕底部滑入，而不是水平。



> MaterialPageRoute对于不同平台，定义了不同的路由动画效果。
>
> - 对于Android，当打开新页面时，新的页面会从屏幕底部滑动到屏幕顶部；当关闭页面时，当前页面会从屏幕顶部滑动到屏幕底部后消失，同时上一个页面会显示到屏幕上。
> - 对于iOS，当打开页面时，新的页面会从屏幕右侧边缘一致滑动到屏幕左边，直到新页面全部显示到屏幕上，而上一个页面则会从当前屏幕滑动到屏幕左侧而消失；当关闭页面时，正好相反，当前页面会从屏幕右侧滑出，同时上一个页面会从屏幕左侧滑入。

**如果想自定义路由动画，可以继承 PageRoute 来实现。**


<br/>

### Navigator

**Navigator** 是一个路由导航组件，提供了打开和退出路由的方法，Navigator 内部通过栈来管理活动路由集合。通常当前屏幕显示的页面就是栈顶路由。**Navigator** 提供了一系列方法来管理路由栈。

**Navigator** 类中第一个参数为 **context** 的静态方法 都对应着一个 Navigaor 的实例方法.比如:

```dart
Navigator.push(BuildContext context,Route route)
 //等同于
Navigator.of(context).push(Route route)
```

#### 常用 api

- **push**

  将给定的路由入栈，即打开新的页面，返回值是一个 Future 对象，用以接收新路由出栈(即关闭)时的返回数据。

- **pop**(BuildContext context,[ result ])

  将栈顶路由出栈，**result** 页面关闭返回给上一个页面的数据
  
- **maybePop**

  判断页面是否可以返回上一页，如果可以直接返回，否则什么都不做；

- **canPop**  

  判断是否可以返回上个页面；

- **removeRoute**

  表示从Navigator中 删除路由，同时释放Route自身资源，路由的生命周期结束；

- **removeRouteBelow**

  表示从Navigator 中删除指定路由下的路由，同时释放其资源，比如 A->B->C，路由栈存在三个页面, 当前处于C，传入B，则删除A，并释放其资源 ；

- **replace**

  将Navigator中指定的路由替换为新的路由；

- **replaceRouteBelow**

  将Navigator中指定的路由下的路由替换为新的路由。比如A-B-C，路由栈中存在三个页面，此时处于C,传入C，则替换B页面为指定新路由页；

#### 示例

<img src="//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad1b7dc34f3344f485090a3fd2b0034c~tplv-k3u1fbpfcp-zoom-1.image" alt="Oct-24-2020 23-03-15" style="zoom:50%;" />



<br/>




<br/>

### 路由传值

用于在路由跳转时携带一些参数，比如打开某个新闻详情页时，我们需要携带 新闻id,这样才能具体知道显示什么。

#### 示例

**接收端**

```dart
 onPressed: () async {
            var result = await Navigator.push<String>(context,
                MaterialPageRoute(builder: (context) {
              return xxxWidget();
            }));
   					//result即为回传的数据
 },
```

**发送端**

```dart
 Navigator.of(context).pop("我是返回的数据");
```


<br/>



## 参考资料

Flutter实战-书籍