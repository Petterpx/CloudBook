# Android四大组件

### Server

> 1. 服务类型

按运行地点，按方法，按运行类型(前台，后台)



> 1. start和bind 启动服务，绑定服务
>
> 2. onStartCommand返回值的用法？
>
>    一共有四种返回值。
>
>    1. **START_STICKY**
>
>       Service被意外终止时不调用 onDestory() 回调，并且终止后自动重启Service生命周期方法。
>
>    2. **START_NOT_STICKY**
>
>       Service被异常终止时不调用 onDestory() 回调，并且不自动重启服务。
>
>    3. **START_REDELIVER_INTENT** 
>
>       Service重新创建并启动，依次回调 onCreate,onStartCommand，并且会把最后依次传给此服务的 intent 重新发给StartCommand
>
>    4. **START_STICKY_COMPATIBILITY**  START_STICKY的兼容类型





### Activity 

启动模式应用场景。





### ContentProvider

内容提供者

进程间通信。

![img](E:\Android_NoteBook\Android_NoteBook\assets\944365-3c4339c5f1d4a0fd.png)

#### 内部原理：

ContentProvider 底层采用 Binder机制。

用于为不同的应用之间共享数据，并提供统一的接口。ContentProvider通过uri来标识其他应用要访问的数据，通过ContentResolver的增删改查方法实现对共享数据的操作。还可以通过注册 ContentObserver 来监听数据是否发生了变化。

