# Flutter入门-Dart线程模型及异常捕获



## Dart单线程模型

在Java和 oc(Objective-C)中，如果程序发生异常且没有被捕获，那么程序将会终止，但是这在 Dart或Js中则不会，原因是这和它们的运行机制相关。Java 和 oc都是多线程模型的编程语言，任意一个线程触发异常且该异常未被捕获时，就会导致整个进程退出。但dart和js不会，他们都是单线程模型，运行机制很相似，但有区别。如果所示：

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk32d3rmwpj30wy0to77p.jpg" alt="image-20201026213733838" style="zoom:50%;" />

Dart在单线程中是以消息循环机制来运行的，其中包含两个任务队列，一个是 **微任务队列-microtask queue**，另一个叫做**事件队列 event queue**.如图所示，微任务队列的执行优先级高于事件队列。

dart线程运行过程，如上图所示，入口函数执行完后，消息循环遍启动了。首先会按照先进先出的顺序遂个执行微任务队列中的任务，时间任务执行完毕后程序便会退出，但是，在事件任务执行的过程中也可以插入新的微任务和事件任务，在这种情况下，整个线程的执行过程便是一直在循环，不会退出，而Flutter中，主线程的执行过程正是如此，永远不会终止。

在Dart中，所有的外部事件都是在事件队列中，如IO，计时器，点击，以及绘制事件等，而微任务通常来源于 Dart 内部，并且微任务非常少，之所以如此，是因为微任务队列优先级高，如果微任务太多，执行时间总和就越久，时间队列任务的延迟也就越就，也就是程序越卡(可以理解为，程序占用主线程时间)。所以得保证微任务队列不会太长，值得注意的是，我们可以通过 **Future.microtask()** 方法向微任务队列插入一个任务。

**在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是当前任务的后续代码就不会执行了，也就是说一个任务中的异常是不会影响其他任务执行。**



## Flutter异常捕获

> 和其他编程语言一样，Dart也可以通过 **try/catch/finally** 来捕获代码块异常。

Flutter框架为我们在很多关键的地方进行了异常捕获，比如，当我们布局发生越界或不合规范时，Flutter就会自动弹出一个错误界面，这是因为Flutter已经在执行 **build** 方法时添加了异常捕获。如图所示

![image-20201026220025443](https://tva1.sinaimg.cn/large/0081Kckwly1gk330oqvinj31b00u0qf9.jpg)

具体发生异常时 Flutter的处理方式，弹出一个 **ErrorWidget**.

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk332j3dnwj30ma0c375m.jpg" alt="image-20201026220212265" style="zoom:150%;" />

如上图所示，我们不难发现，错误最终是通过 **FlutterError.reportError** 方法上报，查看其源码如下：

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk33612iixj30kt074js3.jpg" alt="image-20201026220533814" style="zoom:150%;" />

![image-20201026220513497](https://tva1.sinaimg.cn/large/0081Kckwly1gk335ogyjuj30t00240sv.jpg)

最终，我们发现 **onError**  只是 **FlutterError** 的一个静态属性，并且它有一个默认的处理方法 **dumpErrorToConsole(presenError只是其dumpError..方法通过闭包赋值所产生的静态变量)**，如果我们想自己收集异常，只需要提供一个自定义的错误处理回调接口。如下：

```dart
/// 初始化全局异常 */
void initError() {
  FlutterError.onError = (details) {
    print(details.exceptionAsString());
  };
}
```



## 其他异常捕获与日志收集

在Flutter中，还有一些Flutter 没有为我们捕获的异常，如调用空对象方法异常，Future 中的异常。在Dart 中，异常分两类：同步异常和异步异常，同步异常可以通过 **try/catch** 捕获，而异步异常则相对比较麻烦，例如如下代码无法捕获 **Future** 异常。

```dart
/// 无法捕获的异常
///  在Flutter中，没有private类似的修饰符，如果此方法或者变量不想被其他文件调用，使用 _ 前缀即可
/// */
_noTryError() {
  try {
    Future.delayed(Duration(seconds: 1)).then((value) => Future.error("无法捕获的异常"));
  } catch (e) {
    print(e);
  }
}
```



Dart 中有一个 **runZoned(...)** 方法,可以给执行对象指定一个 Zone, zone表示一个代码执行的环境范围，可以理解为相当于一个 **代码执行沙箱，不同沙箱的之间是隔离的，沙箱可以捕获，拦截或修改一些代码行为，如Zone中可以捕获日志输出，Timer创建，微任务调度的行为，同时Zone 也可以捕获所有未处理的异常。**

![image-20201026230758893](https://tva1.sinaimg.cn/large/0081Kckwly1gk34yz0j0uj30n404dq3j.jpg)

- **zoneValues** :Zone的私有数据，可以通过示例 **zone[key]** 获取，可以理解为每个 **沙箱** 的私有数据。

- **zoneSpecification:** Zone的一些配置，可以自定义一些代码行为，比如拦截日志输出行为。比如：

  **拦截应用中所有调用 print 输出日志的行为：**

  ```dart
  main() {
    runZoned(() => runApp(xxx), zoneSpecification: ZoneSpecification(
        print: (self, parent, zone, line) {
          parent.print(zone, "测试拦截 print 输出：$line");
        }
    ));
  }
  ```

  通过这种方式，我们就可以在应用中记录日志，等到应用触发未捕获的异常时，将异常信息和日志统一上报。当然 Zonespecifiction 还可以自定义一些其他行为。

- **onError：** Zone中未捕获异常处理回调，如果开发者提供了 onError 回调或者通过 **ZoneSpecification.handleUncaughtError** 指定了错误处理回调，那么这个 zone 将会变成一个 error-zone,该 error-zone 中发生未捕获异常(无论同步还是异步)时都会调用开发者提供的回调。如：

  ```dart
  _runZonedError(){
    runZoned((){
      runApp(Text("data"));
    },onError: (Object object,StackTrace stack){
      print(stack.toString()+object.toString());
    });
  }
  ```

这样一来，结合上面的 **FlutterError.onError** 我们就可以捕获Flutter应用中全部错误，需要注意的是，error-zone内部发生的错误不会跨越当前 error-zone的边界，如果想跨越 error-zone 边界去捕获异常，可以通过共同的 **源** zone 来捕获：



## 总结

```dart
/// 收集日志 */
void collectLog(String log) {
  logger.info(log);
}

/// 上传错误和具体的日志逻辑 */
void reportErrorAndLog(FlutterErrorDetails details) {
  logger.info(details.exceptionAsString());
}

/// 构建错误信息 */
FlutterErrorDetails makeDetails(Object object, StackTrace stackTrace) {
  return FlutterErrorDetails(exception: object, stack: stackTrace);
}

/// 示例Main */
void main(){
  FlutterError.onError = (FlutterErrorDetails details) {
    reportErrorAndLog(details);
  };
  runZoned(
        () => runApp(Text("示例")),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
        // 收集所有通过print打印的日志
        collectLog(line);
      },
    ),
    //捕获所有发生的异常
    onError: (Object obj, StackTrace stack) {
      var details = makeDetails(obj, stack);
      reportErrorAndLog(details);
    },
  );
}
```

