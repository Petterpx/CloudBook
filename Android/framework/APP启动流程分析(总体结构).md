# APP启动流程分析(简单了解)

最近面试时碰到了这个问题，自己太菜了，没有答出来。其实这个问题其实涉及挺深的，应该算

Android进阶，在以前的学习中，也没有太关注这方面,毕竟不能一口吃个大胖子。刚好面试碰到了，这次就做个简单记录，先至少有个概念上的认识。

 ![image-20190907102633139](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qqsv55acj31670u0arx.jpg)

### 启动流程：

1. 点击桌面APP图标，Launcher进程采用Binder IPC向 system_server 进程发起 startAvtivity 请求；
2. System_sserver 进程收到请求后，向 zygote进程发送创建进程的请求；
3. Zygote 进程 fork 出新的子进程，即APP进程。
4. APP 进程，通过Binder IPC向 system_server进程发起 attachApplication 请求；
5. system_server进程在收到请求之后，进行一系列准备工作后，在通过 binder IPC向APP进程发送 scheduleLaucheActivity请求；
6. APP 进程的binder线程（ApplicationThread）在收到请求后，通过Handler 向主线程发送LAUNCH_ACTIVITY消息。
7. 主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCrate()等方法。
8. 到此，APP便正式启动，开始进入Activity生命周期,执行完onCrate/onStart/onResume方法，UI渲染结束后便可以看到APP的主界面。

上面的一系列步骤简单介绍了一个APP启动到主页面显示的过程，可能这些流程中的一些术语看的有些懵，什么是Launcher，什么是zygote,什么是 applicationnThead...

下面我们一一介绍一下。



### 理论

#### zygote

zygote意为”受精卵“，Android是基于linux,而在Linux中，所有的进程都是由int 进程直接或则间接 fork 成员来的，zygote进程也不例外。

在Android系统里面，zygote是一个进程的名字。Android是基于 Linux System的，当你的手机开机的时候，Linux 的内核加载完成之后就会启动一个 叫 ”init“ 的进程。在 Linux System 里面，所有的进程都是 由 init 进程fork 出来的，我们的 zygote 进程也不例外。

我们都知道，每一个 app 其实都是

- 一个单独的 dalvik 虚拟机
- 一个单独的进程

所以当系统里面的 第一个 zygote 进程运行之后，在这之后再开启APP，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android系统开启新进程的方式，是通过 fork 第一个 zygot额进程实现的。所以说，出了第一个 zygote 进程，其他应用所在的进程都是 zygote 的子进程。所以它就相当于一个受精卵一样，可以快速分裂，并且产生遗传物质一样的细胞。

#### system_server

SystemServer 也是一个进程，而且是由 zygote 进程fork出来的。

知道了 SystemServer 的本质，我们对它就不算太陌生了，这个进程是Android Framework里面量大非常重要的进程之一-另外一个进程就是上面的zygote进程。

为什么说 SystemServer非常重要呢？因为系统里面重要的服务都是在这个进程里面开启的，比如 ActivityManagerServer,PackageManagerService,WindowManagerService等等。



#### ActivityManagerService

ActivityManagerService，简称AMS，服务端对象，负责系统中所有Activity的生命周期。

ActivityManagerService进行初始化的时机很明确，就是在SystemServer进程开启的时候，就会初始化ActivityManagerService。



**下面我们来看看Android系统里面的服务器和客户端的概念。**

其实服务器客户端的概念不仅仅存在于 Web 开发中，在Android 的框架设计中，使用的也是这一种模式。服务器端指的就是所有APP共有的系统服务，比如我们这里提到的ActivityManagerService,和前面提到的 PackageManageService,WindowManagerService等等，这些基础的系统服务是被所有的APP公有的，当某个app想实现某个操作的时候，要告诉这些系统服务，比如你想打开一个APP，那么我们知道了包名和MainActivity类名之后就可以打开。

```java
Intent intent = new Intent(Intent.ACTION_MAIN);  
intent.addCategory(Intent.CATEGORY_LAUNCHER);              
ComponentName cn = new ComponentName(packageName, className);              
intent.setComponent(cn);  
startActivity(intent);
```

但是，我们的APP通过调用 startActivity() 并不能直接打开另外一个APP，这个方法会通过一系列的调用，最后还是告诉AMS说："我要打开这个APP，我知道他的住址和名字，你帮我打开吧"，所以AMS来通知 zygote 进程来fork一个新进程，来开启我们的目标APP的，这就像浏览器想要打开一个超链接一样。浏览器把网页地址发送给服务器，然后还是服务器吧需要的资源文件发送给客户端的。

知道了 Android Framework 的客户端服务器架构之后，我们还需要了解一件事情，那就是我们的 APP和 AMS（SystemServer进程）还有zygote进程分属于三个独立的进程，他们之间如何通信呢？

**APP 与 AMS 通过Binder 进行IPC 通信，AMS(SystemServer进程)与 zygote通过 Socket进行IPC通信。后面具体介绍。**

那么 AMS 有什么用呢？**在前面我们知道了，如果想打开一个App的话，需要AMS去通知 zygote进程，除此之外，其实所有的 Activity 的开启，暂停，关闭都需要 AMS来控制，所以我们说，AMS负责系统中所有Activity的生命周期。**

在Android系统中，**任何一个Activity的启动都是由AMS和应用程序进程(主要是ActivityThread)相互配合来完成的。AMS服务统一调度系统中所有进程的Activity启动，而每个Activity的启动过程则是由其所属的进程具体来完成。**



#### Launcher

当我们点击手机桌面上的图标时，APP就由Laucher开始启动了。但是，Laucher到底是一个什么东西呢？

**Launcher本质上也是一个应用程序，和我们APP一样，也是继承自Activity.**

```java
public final class Launcher extends Activity
        implements View.OnClickListener, OnLongClickListener, LauncherModel.Callbacks,
                   View.OnTouchListener {
                   }
```

Launcher 实现了点击，长按等回调接口，来接受用户的输入。既然是普通的 APP，那么我们的开发经验在这里就适用，比如，我们点击图标的时候，是怎么开启应用的呢？**捕捉图标点击事件，然后startActivity()发送相应的Intent请求。**



#### Instrumentation 和 ActiivtyThread

每个Activity 都持有Instrumentation对象的一个引用，但是整个进程只会存在一个 Instrumetation对象。Instrumentation这个类里面的方法大多数和 application和Activity有关，**这个类就是完成对Application和Activity初始化和生命周期的工具类**。Instrumentaion 这个类相当于Acivity的管家。

ActivityThread,依赖于UI线程，APP和 AMS 都是通过 Binder 传递信息的，那么ActivityThread就是专门与AMS的外交工作的。



#### ApplicationThread

前面我们知道了App的启动以及Activity的显示都需要AMS的控制，那么我们便需要和服务端的沟通，而这个沟通是双向的。

##### 客户端--》服务端

![image-20190907130235869](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qvb6yo86j317c0akt9r.jpg)

而且由于继承了同样的公共接口类，ActivityManagerProxy提供了与ActivityManagerService一样的函数原型，使用户感觉不出Server是运行在本地还是远端，从而可以更加方便的调用这些重要的系统服务。

##### 服务端--》客户端

还是通过Binder通信，不过是换了另外一对，换成了ApplicationThread和ApplicationThreadProxy。

![image-20190907130500634](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qvdpholhj316s0agdgx.jpg)

他们也都实现了相同的接口 IApplicationThread

```java
  private class ApplicationThread extends ApplicationThreadNative {}

  public abstract class ApplicationThreadNative extends Binder implements IApplicationThread{}

  class ApplicationThreadProxy implements IApplicationThread {}

```



### 启动流程

下面我们详细的说一下启动流程。

#### 创建进程

- 先从Laucher的 startActivity() 方法，通过Binder通信，调用 ActivityManagerService的 startActivity方法。

- 一系列操作，最后调用 startProcessLocked() 方法来创建新的进程。

- 该方法会通过前面讲到的 socket通道传递参数给 Zygote进程。Zygote 孵化自身。调用 ZygoteInit.main() 方法来实例化 ActivityThread对象并最终返回新进程的 pid.

- 调用 ActivityThread.main() 方法，ActivityThread随后依次调用 Looper.prepareLoop() 和 Looper.loop() 来开启消息循环。

  **方法调用流程图如下：**

  ![image-20190907131605267](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qvp8912mj318c0qg7gn.jpg)

更直白的流程解释

![image-20190907133141462](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qw5hv0edj317y0pqqbz.jpg)

1. app发起进程，当从桌面启动应用，则发起进程便是Launcher 所在进程；当从某APP内启动远程进程，则发送进程便是该App所在进程。发起进程先通过Binder 发送消息给system_server进程。
2. system_server进程：调用Process.start() 方法，通过socket 向zygote进程发送创建新进程的请求。
3. zygote进程：在执行ZygoteInit.main() 后便进入 runSelectLoop() 循环体内，当有客户端连接时便会执行 ZygoteConnection.runOnce()方法，再经过层层调用 fork出新的应用进程；
4. 新进程：执行hanlechildProc 方法，最后调用 ActivityThread.main()方法。



#### 绑定Application

上面创建进程后，执行ActivityThread.main() 方法，随后调用attach()方法。

将进程和指定的Application 绑定起来。这个是通过上节的 ActivityThread 对象中调用 bindApplication() 方法完成的。该方法发送一个 BIND_APPLICATION 的消息到消息队列中，最终通过 handleBindApplication() 方法处理该消息，然后调用 makeApplication() 方法来加载 App 的classes到内存中。

**方法调用流程图如下**：

![image-20190907134011132](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qweawq48j317m0mwk2v.jpg)

**更直白的流程解释：**
![image-20190907134041198](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qwetspx2j30vv0u0e02.jpg)

#### 显示Activity界面

经过前两个步骤之后，系统以及拥有了该application 的进程。后面的调用顺序就是普通的从一个已经存在的 进程中启动一个新进程的 activity了。

实际调用方法是 realStartActivity(),它会调用application线程对象中的 scheduleLauchchActivity() 发送一个 LAUNCH_ACTIVITY消息到消息队列中，通过handleLaunchActivity()来处理消息。在handleLaunchActivity()通过performLaunchActivity()方法回调 Activity的onCreate()方法和 onStart()方法，然后通过 handlerResumeActivity() 方法，回调Activity的 onResume() 方法，最终显示Activity界面。

![image-20190907205240427](https://tva1.sinaimg.cn/large/006y8mN6ly1g6r8wccoclj31960fgn4o.jpg)

简单理解：

![image-20190907205305894](https://tva1.sinaimg.cn/large/006y8mN6ly1g6r8wr3rhhj30x10u0jvm.jpg)



#### Binder通信

![image-20190907205349432](https://tva1.sinaimg.cn/large/006y8mN6ly1g6r8xk74h3j316w0seaqj.jpg)

简称：

**ATP**:  ApplicationThteadproxy

**AT:** ApplicationThread

**AMP:** ActivityManagerProxy

**AMS:** ActivityManagerService

图解：

1. sysetm_Server进程中调用startProcessLocked方法，该方法最终通过 socket方式，将需要创建新进程的消息告知 Zygote进程，并阻塞等待Socket返回新创建进程的 pid.
2. Zygote 进程接收到 system_server发送过来的消息，则通过fork的方法，将zygote自身进程复制生成新的进程，并将ActivityThread相关的资源加载到新进程 app process,这个进程可能是用于承载activity等组件。
3. 在新进程app process向servicemanager 查询 system_server进程中binder服务端AMS，获取相对应的Client端，也就是AMP。有了这一对binder c/s 对，那么app process便可以通过 binder 向跨进程 system_server发送请求，即attachApplication()
4. System_server 进程接收到相应binder操作后，经过多次调用，利用 ATP向 app proces发送binder 请求，即 bindApplication.system_server 拥有 ATP/AMS,每一个新创建的进程都会有一个相应的 AT/AMP ，从而可以跨进程进行相互通信。这便是进程创建过程的完整生态链。