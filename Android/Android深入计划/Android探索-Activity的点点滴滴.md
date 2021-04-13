# 基础系列-Activity的点点滴滴

![image-20200425122309961](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5wbc62ehj31440u00y2.jpg)

作为一个老生常谈的话题，Activity是我们接触Android开发遇到的第一个容器，作为一个一年开发经验的小生，结合 Android艺术探索 及我个人的一些理解，简单总结一下Activity相关的一些基础知识。



总体的脑图如下：

![image-20200429192417018](https://tva1.sinaimg.cn/large/007S8ZIlly1geauyqulf4j310w0lzq60.jpg)



## Activity生命周期

基础的生命周期方法这里就不做解释了，大家想必都知道。

![image-20200429192956795](https://tva1.sinaimg.cn/large/007S8ZIlly1geav4n0v4uj30u00z9the.jpg)



- ### **onStart 和 onResume，onPuse 和 onStop有什么实质的不同吗？**

> ***实际使用来说，他们看起来的确差不多，但是 onStart和onStop 是从Activity是否可见这个角度来回调的，而 onResume 和 onPause是从Actvity是否位于前台这个角度来回调的，除了这两点，实际使用中并无其他区别。***

- ### 假设当前 Activity为A，如果这时用户打开一个新的Activity B，那么B的onResume和A的onPuase那个先执行？

  > ***A的 onPause 先执行。***
  >
  > ***在Android的官方文档中，在旧的Activity onPause执行完之后，新的Activity 才能onResume，所以我们应该尽量避免在 onPause 中做太多耗时操作，尽量应该放到onStop中。***



## 异常情况下的生命周期与处理方式

在我们开发中，经常会遇到转屏的问题，而转屏一般也会带来 Activity的重新创建，所以大多数开发者开发的时候，Activity默认是禁止转屏的，但是在一些短视频软件上，转屏就是一件非常常见的事了，那么如何处理相应数据的保存与恢复就是我们必须关注的事了。

### Activity异常下的生命周期图

![image-20200429170808413](https://tva1.sinaimg.cn/large/007S8ZIlly1gear12xtx2j30kr0emjs1.jpg)

当系统配置发生改变后，Activity会被销毁，其 onPause，onStop，onDestory均会被调用，同时由于Activity是在异常情况下终止的，系统会调用 onSaveInstanceState 来保存当前 Activity 的状态。这个方法的调用时机是在 onStop 之前，它和onPause 没有既定的时序关系，有可能在onPause之前调用，也有可能在 onPause之后调用。但需要注意的是，这个方法只会出现在 Activity 被异常终止的情况下。正常情况下不会回调这个方法。

当Actiivty 被重新创建后，系统会调用 onRestoreInstanceState, 并且吧 Activity 销毁时 onSaveInstanceState 方法保存的 Bundle 对象作为参数同时传递给 onRestoreInstanceSate 和 onCreate 方法。因此我们可以通过 onRestoreInstanceState和 onCreate 方法来判断 Activity是否被重建了，如果被重建了，那么我们就可以去除之前保存的数据并恢复，从时序上来说，onRestoreInstanceState 的调用时机在 onStart之后。



### onSaveInstanceState与ViewModel

在上面我们知道，当Activity因为异常情况发生重建时，系统会主动调用 ***onSaveInstanceState*** 方法来进行保存，但需要注意的是 **onSaveInstanceState 并不适合于保存大量数据，Google的推荐是用其来保存相应的id及key,而相应的大量数据推荐使用ViewModel进行保存。**

#### 既然使用ViewModel进行保存了，那 ***onSaveInstanceState*** 的意义还有什么？

> ***onSaveInstanceState*** 是为了保存应用进程在后台时候由于内存限制而被终止，或者配置更改时的回调，其 默认实现是保存了关于 activity的视图层次状态的临时信息，比如EditText中输入的文本，Rv,Lv的滑动位置等，其支持的类型只是Bundle,所以并不适合存储大量数据，适合于少量的临时数据。
>
>  ***ViewModel*** 可以代理复杂数据的加载，也可以作为临时的存储位置，但是不能在手动 ***finish*** 的进程中存留，它的意义更多的是实现 当系统状态更改时，实现数据的保留，而不是ui状态的保留。





## Activity的启动模式 

### 为什么Activity需要启动模式呢？

在默认情况下，当我们多次启动同一个Activity 时，系统会创建多个实例并把他们按照 后进先出的原则(栈结构) 一一放进任务栈中。任务栈是一种 **后进先出** 的栈结构。每次返回时都会销毁一个 Activity 。所以为了更好的管理Activity 和对Activity进行利用，Android提供了 启动模式来解决这个问题，分别为**standard**,**SingleTop**,**singleTask**,**SingleInstance**

### standard

标准模式，也是默认模式。即每次启动一个Activity 都会重新创建一个新的实例，不论这个实例是否存在。相应的生命周期也遵从标准的生命周期过程。

### singleTop

栈顶复用模式。简单理解为，如果新的Activity采用这个模式启动，如果此Activity已经处于当前任务栈栈顶，那么此Activity不会被重复创建，当调用 startActivity跳转时，会回调它的 onNewIntent() 方法。通过此方法我们可以取出当前请求的信息。需要注意的是如果使用 startActivityForResult 跳转，将忽略启动模式。

### singleTask

栈內复用模式。这是一种单实例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会创建实例，和 singleTop 一样，系统也会回调 onNewIntent. 简单理解为，当启动一个 启动模式为 singleTask的Activity时，系统会再栈里寻找是否存在此Activity,如果找到，将此Activity顶部的所有Activity全部出栈，并把其调到栈顶并调用它的 onNewIntent 方法。如果不存在此Activity，则创建此Activity并压入栈顶。

### singleInstance

单实例模式,又称加强的 singTask 模式。它除了具有singleTask的所有特性外，还具有额外的特性，那就是 具有此启动模式的Activity 只能单独位于一个 任务栈中。



## Activtiy的Flags

Activity的Flags有很多，这些标记位在我们实际开发中帮助很大，其中有些标记位可以设定 Activity的启动模式，比如 使用 Application 启动Activity时 添加的 FLAG_ACTIVITY_NEW_TASK 等。

#### FLAG_ACTIVITY_NEW_TASK

为Activity指定 singleTask 的启动模式

#### FLAG_ACTIVITY_SINGLE_TOP

为Activity指定 singleTop 的启动模式

#### FLAG_ACTIVITY_CLEAR_TOP

销毁目标 **Actiivty** 和它之上的所有Activity并重新创建。如果搭配 **FLAG_ACTIVITY_SINGLE_TOP** 使用，会销毁它之上的所有Activity,如果目标实例存在，则不会销毁，会调用其的 ***onNewIntent*** 方法。

#### FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

具有这个标记的 Activity不会出现在历史Activity列表，等同于 xml中指定Activity的属性 android:excludeFromRecents="true"。

**注意，这个参数如果放在首个Activity，那么接下来这个Activity栈的所有Activity都会受到影响。**



## IntentFilter的匹配规则

启动一个Activity分为两种，显式和隐式，日常开发中我们用的最多的就是显式，而隐式常常用作，h5打开一个app，或者调用系统某个设置页，打开相机等等。隐式调用相比显式调用来说，稍微复杂一点，它需要Intent能够匹配目标组件IntentFilter 中所设置的过滤信息，如果不匹配将无法启动目标Activity。 IntentFilter 中的过滤信息 有 **action**,**category**,**data**

### action的匹配规则

action是一个字符串，系统预定义了一些action，同时我们也可以定义我们自己的action。action的匹配规则是Intent中的action必须能够和过滤规则中的action匹配。而过滤规则我们可以定义多个，只要Intent中的action能与任意一个过滤规则匹配就是匹配成功。如果你的Intent中没有定义action，则匹配失败。

```xml
<action android:name="android.intent.action.VIEW" />
```

***常用的action如下：***

- ACTION_MAIN  android.intent.action.MAIN   应用程序入口  
- ACTION_VIEW  android.intent.action.VIEW  显示数据给用户  
- ACTION_ATTACH_DATA  android.intent.action.ATTACH_DATA  指明附加信息给其他地方的一些数据 
- 
  ACTION_EDIT  android.intent.action.EDIT  显示可编辑的数据    
- ACTION_PICK  android.intent.action.PICK  选择数据    
- ACTION_CHOOSER  android.intent.action.CHOOSER  显示一个Activity选择器   
- ACTION_DIAL  android.intent.action.GET_CONTENT  显示打电话面板    
- ACITON_CALL  android.intent.action.DIAL  直接打电话    
- ACTION_SEND  android.intent.action.SEND  直接发短信    
- ACTION_SENDTO  android.intent.action.SENDTO  选择发短信    
- ACTION_ANSWER  android.intent.action.ANSWER  应答电话    



### category匹配规则

category是一个字符串，系统也为我们预制了一席，对于在 已经定义的匹配规则，在Intent 中存在的categoty必须全部符合已经定义了的规则，当然也可以不填，如果Intent中没有包含，系统会为我们默认带上 **android.intent.category.DEFAULT**，这一点和action 有一些区别.

```xml
<category android:name="android.intent.category.DEFAULT" />
```



### data 匹配规则

data的匹配规则和 action 类似，如果定义了相应的匹配规则，那么Intent中必须包含相应的data.

```xml
  <data
        android:host="*"
        android:mimeType="string"
        android:path="/test"
        android:pathPattern="*"
        android:pathPrefix="/petterp"
        android:port="1080"
        android:scheme="http" />
```

data由两部分组成，mimeType和URI,mimeType指媒体类型，比如image/jpeg.  Audio/mpeg4-genric和 video/*等，可以表示图书，文本，视频等不同的媒体形式。而URI中包含的数据就相对多一点。

#### Uri的结构如下：

```
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
```

- **Scheme**: URI 的模式，比如 http,file,content等，如果 URI中没有指定 scheme.那么整个 URI的其他参数无效，这也意味着URI是无效的。
- **Host**: URI的主机名，比如 www.baidu.com.如果host未指定，那么整个Uri中的其他参数无效，这也意味着URI是无效的。
- **Port**：URI中的端口号，比如 80，仅当URI 中指定了 scheme 和 host参数的时候，port参数才是有意义的。Path,pathPattern和 pathPrefix这三个参数表示路径信息。
  - 其中path表示完整的路径信息，
  - pathPattern 也表示完整的路径信息，但是它里面可以包含通配符  ***** ，***** 表示0个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想要表示真实的字符串，那么 ***** 要写成  \\\\*  ,\ 要写成 \\\\\ 
  - pathPrefix表示路径的前缀信息。  

#### data的匹配规则

> data的匹配规则和action 类似，它也要求 Intent 中必须含有 data数据，并且 data数据能够完全匹配过滤规则中的某一个 data,这里的完全匹配是指 过滤规则中出现的 data部分也出现在了 Intent 中的 data中。

如下所示匹配规则：

```xml
    <intent-filter>
      					<action android:name="petterp.test1" />
                <data android:mimeType="image/*" />
     </intent-filter>
```

这种匹配规则指定了媒体类型为所有类型的图片，那么Intent中的 **mineType** 属性必须为 ***image/**** 才能匹配，这种情况下虽然过滤规则没有指定URI，但是却有默认值。URI 的默认值为 **content** 和 **file** 。也就是说，虽然没有指定 URI，但是Intent中的 URI 部分的 **schema** 必须为 **content** 或者 **file**才能匹配。

所以如果我们要调用的话，可以写出如下的代码：

```xml
val intent=Intent("petterp.test1")
intent.setDataAndType("file://abc","iamge/*")
```

接着，我们再看一个案例，相比上面的，复杂一点

```xml
	<intent-filter>
                <action android:name="petterp.test2" />
                <category android:name="android.intent.category.DEFAULT" />
                <data
                    android:host="www.baidu.com"
                    android:mimeType="image/*"
                    android:path="/test"
                    android:port="8080"
                    android:scheme="https"
                    android:pathPattern=".*"/>
  </intent-filter>
```

对于这个案例，我们在调用的时候，就必须采用以下格式：

```kotlin
val intent = Intent("petterp.test2")
intent.setDataAndType(Uri.parse("https://www.baidu.com:8080/test/路径随便"),"image/*")
```

如果这里看不懂的话，多看一遍data匹配规则就行。

***注意：category必须加，就算Intent(intent中默认会带一个)中不加，xml中也必须带，否则会存在找不到相应的Activity***
