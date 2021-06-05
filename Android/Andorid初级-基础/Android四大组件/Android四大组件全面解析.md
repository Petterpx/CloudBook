# Android四大组件
## Activity

### 生命周期

![img](https://upload-images.jianshu.io/upload_images/682504-1405607172778d9b.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/545/format/webp)

### 1.与Fragment进行绑定时的生命周期变动

**SDK28 模拟器28**

> 进入Activity，绑定Fragment，然后点击返回键之后
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019062621242858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)



> 进入Activity，绑定Fragment，点击home，然后重新进入，再点击返回
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190626212525818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)



> 进入Activity，绑定Fragment，点击home，然后直接打开任务管理器kill掉任务
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019062621262796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)
>
>
> 为什么onDestroy没有执行？
>
> 用一句话概述
>
> **不要在onStop，onDestory中保存重要数据；最近任务栏清除app，app的onDestory不会掉用是因为该Binder调用是非阻塞的，导致App被杀死，所以onDestory没来得及调用**
>
> *详细链接参考*
>
> <https://blog.csdn.net/u013122625/article/details/77916482>



### 2.特殊情况下的生命周期

上面是普通情况下Activity生命周期的一些流程，但是在一些特殊情况下，Activity的生命周期的经历有些异常，下面就是两种特殊情况。
**①横竖屏切换**
在横竖屏切换的过程中，会发生Activity被销毁并重建的过程。

在了解这种情况下的生命周期时，首先应该了解这两个回调：**onSaveInstanceState和onRestoreInstanceState**。

在Activity由于异常情况下终止时，系统会调用onSaveInstanceState来保存当前Activity的状态。这个方法的调用是在onStop之前，它和onPause没有既定的时序关系，该方法只在Activity被异常终止的情况下调用。当异常终止的Activity被重建以后，系统会调用onRestoreInstanceState，并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象参数同时传递给onRestoreInstanceState和onCreate方法。因此，可以通过onRestoreInstanceState方法来恢复Activity的状态，该方法的调用时机是在onStart之后。**其中onCreate和onRestoreInstanceState方法来恢复Activity的状态的区别：** onRestoreInstanceState回调则表明其中Bundle对象非空，不用加非空判断。onCreate需要非空判断。建议使用onRestoreInstanceState。

![img](http://upload-images.jianshu.io/upload_images/3985563-23d90471fa7f12d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

横竖屏切换的生命周期：onPause()->onSaveInstanceState()-> onStop()->onDestroy()->onCreate()->onStart()->onRestoreInstanceState->onResume()

可以通过在AndroidManifest文件的Activity中指定如下属性：

```java
android:configChanges = "orientation| screenSize"
```

来避免横竖屏切换时，Activity的销毁和重建，而是回调了下面的方法：

```java
@Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
```

**②资源内存不足导致优先级低的Activity被杀死**
Activity优先级的划分和下面的Activity的三种运行状态是对应的。

(1) 前台Activity——正在和用户交互的Activity，优先级最高。

(2) 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户交互。

(3) 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。

当系统内存不足时，会按照上述优先级从低到高去杀死目标Activity所在的进程。我们在平常使用手机时，能经常感受到这一现象。这种情况下数组存储和恢复过程和上述情况一致，生命周期情况也一样。

### 3.Activity的三种运行状态

①Resumed（活动状态）

又叫Running状态，这个Activity正在屏幕上显示，并且有用户焦点。这个很好理解，就是用户正在操作的那个界面。

②Paused（暂停状态）

这是一个比较不常见的状态。这个Activity在屏幕上是可见的，但是并不是在屏幕最前端的那个Activity。比如有另一个非全屏或者透明的Activity是Resumed状态，没有完全遮盖这个Activity。

③Stopped（停止状态）

当Activity完全不可见时，此时Activity还在后台运行，仍然在内存中保留Activity的状态，并不是完全销毁。这个也很好理解，当跳转的另外一个界面，之前的界面还在后台，按回退按钮还会恢复原来的状态，大部分软件在打开的时候，直接按Home键，并不会关闭它，此时的Activity就是Stopped状态。



####  4.Activity启动模式

> Android的启动模式一共有4种，默认情况我们使用的是标准模式。
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190626212754487.png)
>
> 通过在 <activity> 里添加 **android:launchMode **即可设置相应的启动模式~

- 标准模式（standard）
- 栈顶复用模式（singleTop）
- 栈内复用模式（singleTask）
- 单例模式（singleInstance）



#### 4.1启动模式的结构——栈

Activity的管理是采用任务栈的形式，任务栈采用“后进先出”的栈结构。

![img](http://upload-images.jianshu.io/upload_images/3985563-83df2c2b5d3afafd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 标准模式

> 活动的默认启动模式，简单来说，在你不指定启动模式的情况下，你**每打开一个新的Activity，新的Activity都会加入你的返回栈并且处于栈顶状态。这样当你点返回的时候就是一层一层退栈。遵循后进先出的原则。**

#### 栈顶复用模式

> 有些时候，你会觉得 standard 模式有点别扭，明明已经在栈顶了，如果是相同的Activity还要在启动时创建一次，不麻烦吗。所以singleTop 就可以解决你这个问题，**如果在启动时发现返回栈的栈顶已经是当前Activity，则不会再次创建，而是直接复用。**

#### 栈内复用

> 与栈顶复用不同的是，**singleTask会在启动Activity时会先检查栈内是否有此Activity,如果有，则将这个Activity之前的栈全部退栈，这样要启动的Activity就成了栈顶，免去了重新创建。但是如果当前不存在此Activity,则会创建一个新的Activity来管理此活动。**所以要注意使用时的需求。

#### 单例模式

> 单例模式？设计模式，字名一样，意思从某种程序上来说也差不多，全局唯一。
>
> 与上面三种不同的是，指定为 singleInstance 的模式，**在启动的时候会启用一个新的返回栈来管理此活动，而且只会创建一次(当然，如果你kill掉之后再启动就另当别论啦)，如此一来，全局独立并唯一。**


<br/>
<br/>
<br/>
<br/>

## Service

> Service是Android中实现程序后台运行的解决方案。但是需要注意的是，Service默认不会运行在子线程，它也不允许在一个独立进程中，它同样执行UI操作。因此不要在Service中执行耗时操作。除非Service中创建了子线程来完成耗时操作。

1. IPC: 简称进程间通信，是指两个进程之间进行数据交换的过程。
2. AIDL :用于生成可以在Android设备上两个进程之间进行IPC的代码。如果在一个进程中(比如Activity)要调用另一个进程中(比如Serveice)对象的操作，就可以使用AIDL生成可序列化的参数。

**关于AIDL 及 IPC本篇不会过多涉及。先了解广度即可，在以后的博客里我会逐渐涉及并记录。**

<br/>

### 1. 按运行地点分类：

#### 1.1 本地服务(Local Service)

> **该服务依附于主线程。**
>
> **而不是独立进程，这样在一定程度上节约了资源，另外Local服务因为是在同一进程因此不需要IPC，也不需要AIDL。所以 bindService 会方便很多。**
>
> **需要记住的是 主进程被Kill后(onbindStart)，服务便会终止。**

#### 1.2 远程服务（Remote Service）

1. #### > **该服务是独立进程，对应进程名格式为所在包名加上指定的 android:process 字符串。由于是独立进程，因此在Activity所在进程被kill时，该服务依然运行，不受影响。**,>,> **但需要注意的是：因为该服务是独立进程，会占用一定资源，并且使用 AIDL 进行 IPC会稍微麻烦。**

<br/>

### 2 按运行类型分类

#### 2.1 前台服务：

> **会在通知栏显示 常存的 Notification**
>
> **当服务被终止时，通知栏的 Notification也会消失，对于用于有一定的提醒作用，比如音乐播放器通知栏旁边的 x 号点击之后。(当然这里指的是少数播放器，并不是所有播放类软件都会带)**

#### 2.2 后台服务：

> **默认的服务即为后台服务，即不会在通知栏显示 常存的 Notification**
>
> **服务被终止时用户无法察觉，如天气的更新，数据的同步等等。**

<br/>

### 3 按使用方式分类

#### 3.1 startService启动的服务

> **主要用于启动一个服务执行后台任务，不进行通信。停止服务使用 onstopService.**

#### 3.2 binService启动的服务

> **与服务通信绑定时使用的方法，停止服务使用 unbindService.**

#### 3.3 同时使用startService和bindService启动

> **停止服务应同时使用 stopService 与 unbindservice**

<br/>

### 4 本地服务的启动方式

#### 4.1 第一种

1. 通过start方式开启服务：
2. 使用service的步骤：

> 1. 定义一个类继承 service
> 2. manifest.xml文件中配置 service （当然as一键创建不用配置）
> 3. 使用context的startService（Intent）方法启动服务
> 4. 不使用时，调用stopService(Intent)方法停止服务

使用start方式启动的生命周期

> **onCreate()->onStartCommand()->onDestory();**
>
> **如果服务已经开启，不会重复回调 onCreate() 方法，如果再次调用 startService()方法，service 而是会调用 onstart或者 onStartCommand()。停止服务需要调用 stopService() 方法，服务停止的时候回调 onDestory被销毁。**
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190626212919673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)

#### 4.2 第二种

**采用 bind 的 方式开启服务**

> 1. 定义一个类继承 Service
> 2. 在manifest.xml 文件中注册 service
> 3. 使用 context 的 bindService(Intent,ServiceConnection,int )方法启动Service
> 4. 不再使用时，调用unbindService()方法停止该服务
>     ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190626213017576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)
>
>
> 生命周期，只会绑定一次，当多次调用绑定服务时，只会多次调用 startService()方法

Demo

```java
public class ServiceDemo extends Service {
    private MyBinder binder=new MyBinder();
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("demo","onCreate创建");
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Log.e("demo","解绑");
        return super.onUnbind(intent);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("demo","销毁");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e("demo","onStartCommand服务启动");
        return super.onStartCommand(intent, flags, startId);
    }
    class MyBinder extends Binder {

        private CountDownTimer count;

        public void start(){
            count = new CountDownTimer(Integer.MAX_VALUE,1000) {
                @Override
                public void onTick(long millisUntilFinished) {
                    Toast.makeText(ServiceDemo.this, "启动啦", Toast.LENGTH_SHORT).show();
                }

                @Override
                public void onFinish() {

                }
            }.start();
        }
        public void onstop(){
            if (count != null) {
                count.cancel();
                count=null;
            }
        }
    }
    @Override
    public IBinder onBind(Intent intent) {
        Log.e("Demo","bind");
        return binder;
    }
}
```

```java
public class MyServerConnection implements ServiceConnection {
    private ServiceDemo.MyBinder myBinder;

    /**
     * 第二个参数是Server中的onBind方法返回的。
     *
     * @param name
     * @param service
     */
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Log.e("demo","开始");
        myBinder = (ServiceDemo.MyBinder) service;
        myBinder.start();
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        myBinder.onstop();
    }
}
```

```java
public class Main2Activity extends AppCompatActivity {
    MyServerConnection myServerConnection=new MyServerConnection();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        findViewById(R.id.btn_start).setOnClickListener(v -> {
            Log.e("demo","startService启动服务");
            Intent intent=new Intent(this,ServiceDemo.class);
            bindService(intent,myServerConnection,BIND_AUTO_CREATE);
        });
        findViewById(R.id.btn_stop).setOnClickListener(v -> {
            Log.e("demo","stopService服务关闭");
           unbindService(myServerConnection);
        });
    }
}
```

<br/>



### 5. 远程服务

步骤1：新建定义AIDL文件，并该声明服务需要向客户端的提供的接口
 步骤2：在服务子类中实现AIDL中定义的接口方法，并定义生命周期的方法（onCreat，onBind（），blabla）
 步骤3：在AndroidMainfest.xml中注册服务＆声明为远程服务

**客户端（客户端）**
  步骤1：拷贝服务端的AIDL文件到目录下
 步骤2：使用Stub.asInterface接口获取服务器的活页夹，根据需要调用服务提供的接口方法
 步骤3：通过意图指定服务端的服务名称和所在包，绑定远程服务

参考 链接：https://www.jianshu.com/p/34326751b2c6

<br/>

### 6. IntentService

> IntentService是Service的子类，比普通的Service增加了额外的功能。先看**Service本身存在两个问题**：

- Service不会专门启动一条单独的进程，Service与它所在应用位于同一个进程中；
- Service也不是专门一条新线程，因此不应该在Service中直接处理耗时的任务；

IntentService特征:

- 会创建独立的worker线程来处理所有的Intent请求；
- 会创建独立的worker线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题；
- 所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service；
- 为Service的onBind()提供默认实现，返回null；
- 为Service的onStartCommand提供默认实现，将请求Intent添加到队列中；



<br/><br/>


## BroadcastReceiver-广播

### 1. 分类:

#### 1.1 标准广播（Normal brodcasts）

> 标准广播是完全异步的，可以在几乎同一时刻被所有接受者接受到。因此他们之间没有任何先后顺序科研。这种广播效率比较高，但同时也意味着它是无法被截断的。



#### 1.2 有序广播(Ordered broadcasts)

> 是一种同步执行的广播，在广播发出之后，同一时刻只会有一个广播接收器能够收到这条广播消息，当这个广播接收器中的逻辑执行完毕后，广播才会继续传递。所以此时的广播接收器是有先后顺序的，优先级高的广播接收器就可以先收到广播消息，并且前面的广播接收器还可以截断正在传递的广播。

#### 1.3 本地广播 （Lolcal Brodcast） :

>  本地广播，即只在APP内传播，安全性高。

<br/>

### 2 发送广播

```java
Context.sendBroadcast()
发送的是普通广播，所有订阅者都有机会获得并进行处理。
```

```java
Context.sendOrderedBroadcast()
发送的是有序广播，系统会根据接收者声明的优先级别按顺序逐个执行接收者，前面的接收者有权终止广播(BroadcastReceiver.abortBroadcast())，如果广播被前面的接收者终止，后面的接收者就再也无法获取到广播。对于有序广播，前面的接收者可以将处理结果通过setResultExtras(Bundle)方法存放进结果对象，然后传给下一个接收者，通过代码：Bundle bundle =getResultExtras(true))可以获取上一个接收者存入在结果对象中的数据。
系统收到短信，发出的广播属于有序广播。如果想阻止用户收到短信，可以通过设置优先级，让你们自定义的接收者先获取到广播，然后终止广播，这样用户就接收不到短信了。
```

> 步骤：
> 1，自定义一个类继承BroadcastReceiver
> 2，重写onReceive方法
> 3，在manifest.xml中注册

需要注意的是：**BrodcastReceiver生命周期很短**

如果需要在onReceiver 完成一些耗时操作，应该考虑在Service中开启一个新线程处理耗时操作，不应该在 BrodcastReceiver中开启一个新的线程，**因为BroadcstReceiver生命周期很短，在执行完 onReceiver 以后就结束，如果开启一个新的线程，可能出现 BroadcastRecevier 退出以后线程还在，而如果 BroadcastReceiver 所在的进程结束了，该线程就会被标记为一个空线程，根据 Android 的内存管理策略，在系统内存紧张的时候，会按照优先级，结束优先级低的线程，而空线程无异是优先级最低的，这样就可能导致 BroadcastReceiver启动的子线程不能执行完成。**



### Demo

#### 2.1 静态注册

```java
public class MyReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e("demo", "接收到一条消息");
    }
}
```

```java
public class MainActivity extends AppCompatActivity {
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main4);
        findViewById(R.id.btn_send_Nor).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent("xxx");
                sendBroadcast(intent);
            }
        });
    }
}
```

```xml
//xml
<receiver
    android:name=".BroadcstReceiver.MyReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="xxx"/>
    </intent-filter>
</receiver>
```

<br/>

#### 2.2 动态注册

**AndroidMonifest 一定添加**

```
<uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
```

**Activity**

```java
registerReceiver(new MyReceiver(),new IntentFilter("xxx"));
```

```java
sendBroadcast(new Intent("xxx"));
```

**xml中注册的广播优先级高于动态注册**

<br/><br/>

### 3. 内部实现机制

1. 自定义广播接受者 BrodcastReceiver，并复习 onRecvice()方法
2. 通过 Binder 机制 AMS (Activity Manager Service)进行注册
3. 广播发送者通过 Binder 机制向AMS发送广播
4. AMS查找符合相应条件(IntentFilter/Permission等) 的BriadcastReaceiver，将广播发送到 BrodcastReceiver(一般情况下是Activity)相应的消息循环队列之中。
5. 消息循环 执行拿到此广播后，回调 BrodcastReceiver 中的 onReceiver() ,完成广播发送

<br/>

### 4. 本地广播 

###  4.1 LocalBrodcastManager详解

1. 使用它发送的广播将只在自身app传播，因此不必担心泄漏隐私数据
2. 其他APP 无法对你的app发送该广播，因为你的app 根本就不可能接收到非自身应用发送的该广播，因此不必担心有安全漏洞可以利用。
3. 比系统的全局广播更加高效。

**Demo**

```java
public class Main4Activity extends AppCompatActivity {
    private LocalBroadcastManager localBroadcastManager;
    private IntentFilter intentFilter;
    private Receiver receiver;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main4);
        //过滤器
        intentFilter=new IntentFilter();
        //添加意图
        intentFilter.addAction("xxx");
        receiver=new Receiver();
        //获取实例
        localBroadcastManager=LocalBroadcastManager.getInstance(this);
        //注册本地广播监听器
        localBroadcastManager.registerReceiver(receiver,intentFilter);
        findViewById(R.id.btn_send_Nor).setOnClickListener(v -> {
            Intent intent=new Intent("xxx");
            localBroadcastManager.sendBroadcast(intent);
        });
    }
    class Receiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "收到消息", Toast.LENGTH_SHORT).show();
        }
    }

}
```

**4.2 LocalBroadcastManager**

1. LocalBroadcastManager 高效的原因主要是因为它内部是通过Handler 实现的，它的 senBroadcast() 方法含义并非和我们平常所用的一样，它的 sendBroadcast() 方法其实是通过 Handler 发送了一个 Message 实现的。
2. 既然它内部是通过Handler实现广播发送，那么相比系统广播通过Binder 实现那肯定是更高效了。同时 别的应用无法向我们的应用发送广播，而我们应用内发送的广播也不会离开我们的应用。
3. LocalBroadcastManager 内部协作主要是靠两个 Map ：mReceier 和 mActions ,当然还有一个 List 集合 mPendingBroadcast,这个主要就是存储待接收的广播对象。

<br/>

### 5. 静态注册于动态注册的区别

- 静态广播：

  注册完成就一直在运行  

  直接把广播接受者写在AndrodMofit,即使Activity被销毁，还是可以收到广播。

- 动态注册：必须在代码中执行  受activity的生命周期影响

- 当广播为有序广播时：

  1. 同优先级的广播接收器，静态注册优先级高于动态注册
  2. 同优先级的同类广播接收器，静态广播：先扫描的优先于后扫描的。动态广播：先注册得优先于后注册的。

- 当广播为标准广播时：

  1. 无视优先级，动态广播优先于静态广播接收器
  2. 同优先级的同类广播接收器，静态广播：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。

<br/>

### 6.  需要注意的地方

> 当如果要进行的操作需要花费比较长的时间，则不适合放在BroadcastReceiver中进行处理。
> 引用网上找到的一段解释：
> 在 Android 中，程序的响应（ Responsive ）被活动管理器（ Activity Manager ）和窗口管理器（ Window Manager ）这两个系统服务所监视。当 BroadcastReceiver 在 10 秒内没有执行完毕，Android 会认为该程序无响应。所以在 BroadcastReceiver 里不能做一些比较耗时的操作，否侧会弹出ANR （ Application No Response ）的对话框。如果需要完成一项比较耗时的工作，应该通过发送Intent 给 Service ，由 Service 来完成。而不是使用子线程的方法来解决，因为 BroadcastReceiver 的生命周期很短（在 onReceive() 执行后 BroadcastReceiver 的实例就会被销毁），子线程可能还没有结束BroadcastReceiver 就先结束了。如果 BroadcastReceiver 结束了，它的宿主进程还在运行，那么子线程还会继续执行。但宿主进程此时很容易在系统需要内存时被优先杀死，因为它属于空进程（没有任何活动组件的进程）。


<br/><br/><br/>


## ContentProvider 内容提供者


> Android四大组件之一，它主要作用就是将程序的内部数据和外部进行共享，微数据提供外部访问接口，被访问的数据主要以数据库的形式存在，而且还可以选择共享那一部分的数据。这样一来，对于程序当中的隐私数据可以不共享，从而更加安全。contentprovider是一种跨程序共享数据的重要组件。

![img](http://upload-images.jianshu.io/upload_images/944365-3c4339c5f1d4a0fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



1. ### 为什么Android要提供 ContentProvider ，而不是直接让我们进行操作，这样不是更复杂吗？

> 因为我们一一部手机里面可不只有一个app提供内容，它可能安装了很多含有提供商的应用，比如联系人，日历等。有如此多的提供者，如果你开发一块应用要使用其中多个，你不得了解每个 ContentProvider  的不同实现吗，这样来看，岂不是工作量特别大。所以Android为我们提供了 ContentProvider 来同意管理与不同的 ContentProvider 间的操作。
>
> ![img](https://upload-images.jianshu.io/upload_images/1362430-a336044d52818448.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/836/format/webp)



只需它是靠什么来制定不同的访问规则，请看下面。

![img](http://upload-images.jianshu.io/upload_images/944365-96019a2054eb27cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
// 设置URI
Uri uri = Uri.parse("content://com.carson.provider/User/1") 
// 上述URI指向的资源是：名为 `com.carson.provider`的`ContentProvider` 中表名 为`User` 中的 `id`为1的数据

// 特别注意：URI模式存在匹配通配符* & ＃

// *：匹配任意长度的任何有效字符的字符串
// 以下的URI 表示 匹配provider的任何内容
content://com.example.app.provider/* 
// ＃：匹配任意长度的数字字符的字符串
// 以下的URI 表示 匹配provider中的table表的所有行
content://com.example.app.provider/table/#
```



### 2. 使用方法：

**新建一个类继承ContentProvider的方式，并重写它的6个抽象方法。**

**1.onCreaete()**

> 初始化内容提供器，通常会在这里完成，对数据库的创建和升级数据库，返回true,和false,

**2.query()**

> 从内容提供器中查询数据，使用uri参数确定来查询那个那张表，progjction参数用于确定查询那些列，selection和selectionAargs参数用于约束查询哪些行，查询的结果存放在Cursor对象中。

**3.insert()**

> 想内容提供器中添加一条数据，使用uri参数来确定要添加到的表，待添加的数据保存在values参数中，添加完成后，返回一个用于表示这条新记录的uri.

**4.update()**

> 更新内容提供器中已有的数据，使用URI参数来确定更新那一张表中的数据，新数据保存在values参数中，selection和selectionArgs参数用于约束更新那些行，受影响的的行数将做为返回值返回。

**5.delete()**

> 从内容提供器中删除数据2，使用uri参数来确定删除哪一样表中的数据，selection和selectionArgs参数用于约束删除那些行，被删除的行数将作为返回值返回。

**6.getType（）**

> 根据返回的内容URI来返回相应的MIME类型。

**而他们每一个方法都带有一个uri参数，这个参数正是调用ConterntResolver的增删改查方法时传递过来的。**

更多ContentProvider参考链接

https://lrh1993.gitbooks.io/android_interview_guide/content/android/basis/ContentProvider.html