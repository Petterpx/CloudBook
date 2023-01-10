# 由浅入深，详解ViewModel的那些事

Hi，你好 :)

## 引言

关于 **ViewModel** ，Android 开发的小伙伴应该都非常熟悉，无论是新项目还是老项目，基本都会使用到。而 **ViewModel** 作为 `JetPack` 核心组件，其本身也更是承担着不可或缺的作用。

因此，了解 **ViewModel** 的设计思想更是每个应用层开发者必不可缺的基本功。

随着这两年 `ViewModel` 的逐步迭代，比如 **SaveStateHandle** 的加入等，**ViewModel** 也已经不是最初版本的样子。要完全理解其设计体系，往往也要伴随这其他组件的基础，所以并不是特别容易能被开发者吃透。

故本篇将以最新视角开始，与你一起，用力一瞥 **ViewModel** 的设计原理。

**本文对应的组件版本：**

- *Activity-ktx-1.5.1*
- *ViewModel-ktx-2.5.1*

> 本篇定位中等，将从背景与使用方式开始，再到源码解读。由浅入深，解析 `ViewModel` 的方方面面。

## 导航

学完本篇，你将了解或明白以下内容：

- `ViewModel` 的使用方式；
- `SavedStateHandle` 的使用方式；
- `ViewModel` 创建与销毁流程；
- `SavedStateHandle` 创建流程；

好了，让我们开始吧! 🐊

## 基础概念

在开始本篇前，我们先解释一些基础概念，以便更加清晰的了解后续的状态保存相关。

### 何谓配置变更?

配置变更指的是，**应用在运行时，内置的配置参数变更从而触发的Activity重新创建**。

常见的场景有：旋转屏幕、深色模式切换、屏幕大小变化、更改了默认语言或者时区、更改字体大小或主题颜色等。

### 何谓异常重建？

异常重建指的是非配置变更情况下导致的 `Activity` 重新创建。

常见场景大多是因为 **内存不足，从而导致后台应用被系统回收** ，当我们切换到前台时，从而触发的重建，这个机制在Android中为 `Low Memory Killer` 机制，简称 `LMK`。

> 可以在开发者模式，限制后台任务数为1，从而测试该效果。

## ViewModel存在之前的世界

在 `ViewModel` 出现之前,对于 `View` 逻辑与数据，我们往往都是直接存在 `Activity` 或者 `Fragment` 中，优雅一点，会细分到具体的单独类中去承载。当配置变更时，无可避免，会触发界面重绘。相应的，我们的数据在没有额外处理的情况下，往往也会被初始化，然后在界面重启时重新加载。

但如果当前页面需要维护某些状态不被丢失呢，比如 选择、上传状态 等等? 此时问题就变得棘手起来。

稍有经验同学会告诉你，**在 onSaveInstanceState 中重写，使用bundle去存储相应的状态啊？➡️**

**但状态如果少点还可以，多一点就非常头痛，更别提包含继承关系的状态保存。☹️**

所以，不出意外的话，我们 App 的 **Activity-manifest** 中通常默认都是下列写法:

```xml
android:configChanges="keyboard|orientation|uiMode|..."
```

> 这也是为啥Android程序普遍不支持屏幕旋转的一部分原因，从源头扼杀因部分配置变更导致的状态丢失问题。🐶保命

## VideModel存在之后的世界

随着 `ViewModel` 组件推出之后，上述因配置变更而导致的状态丢失问题就迎刃而解。

`ViewModel` 可以做到在配置变更后依然持有状态。所以，在现在的开发中，我们开始将 **View数据** 与 逻辑 藏于 `ViewModel` 中，然后对外部暴漏观察者，比如我们常常会搭配 `LiveData` 一起使用，以此更容易的保持状态同步。

关于 `ViewModel` 的生命周期，具体如下图所示：

![viewmodel-lifecycle](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202212142355129.png)

虽然 `ViewModel` 非常好用，但 `ViewModel` 也不是万能，其只能避免配置变更时避免状态丢失。比如如果我们的App是因为 **内存不足** 而被系统**kill** 掉，此时 `ViewModel` 也会被清除⚠️。

不过对于这种情况，仍然有以下三个方法可以依然保存我们的状态:

- 重写 `onSaveInstanceState()` 与 `onRestoreInstanceState()`;
- 使用 `SavedState`,本质上其实还是 `onSaveInstanceState()` ；
- 使用 `SavedStateHandle` ，本质上是依托于 `SaveState` 的实现;

> 上述的后两种都是随着 **JetPack** 逐步被推出，可以理解为是对原有的onSavexx的封装简化，从而使其变得更易用。

关于这三种方法，我们会在 `SavedStateHandle` 流程解析中再进行具体叙述，这里先提出来，留个伏笔。

## ViewModel使用方式

作为文章的开始，我们还是要先聊一聊 `ViewModel` 的使用方式，如下例所示:

![image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/image.7etvanmirq00.webp)

> 当然，你也可以选择引入 **activity-ktx** ,从而以更简便的写法去写：
>
> ```groovy
> implementation 'androidx.activity:activity-ktx:1.5.1'
> 
> private val mainModel by viewModels<MainViewModel>()
> ```

示例比较简单，我们创建了一个 `ViewModel` ，如上所示，并在 `MainActivity` 的 onCreate() 中进行了初始化。

这也是我们日常的使用方式，具体我们这里就不再做阐述。

## SavedStateHandle使用方式

我们知道，`ViewModel` 可以处理因为配置更改而导致的的状态丢失，但并不保证异常终止的情况，而官方的 `SavedStateHandle` 正是用于这种情况的解决方式。

`SavedStateHandle` ,如名所示，用于保存状态的手柄。再细化点就是，用于保存状态的工具，从而配合 `ViewModel` 而使用，其内部使用一个 **map** 保存我们要存储的状态，并且其本身使用 `operator` 重载了 **set()** 与 **get()** 方法，所以对于我们来说，可以直接使用 **键值对 ** 的形式去操作我们要保存的状态，这也是官方为什么称 `SavedStateHandle` 是一个 **具有键值映射Map** 特性的原因。

> 在 Fragment1.2 及 Activity1.1.0 之后, `SavedStateHandle` 可以作为 ViewModel 的构造函数，从而反射创建带有 `SavedStateHandle` 的 ViewModel 。

**具体使用方式如下：**

![SavedStateHandle](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/SavedStateHandle.simple.webp)

我们在 `MainViewModel` 构造函数中新增了一个参数 **state:SavedStateHandle** ,这个参数在 `ViewModel` 初始化时，会帮我们自动进行注入。从而我们可以利用 `SavedStateHandle` 以key-value的形式去保存一些 **自定义状态** ,从而在进程异常终止，Act重建后，也能获取到之前保存的状态。

**至于为什么能实现保存状态呢？**

主要是因为 `SavedStateHandle` 内部默认有一个 **SavedStateRegistry.SavedStateProvider** 状态保存提供者对象，该对象会在我们创建`ViewModel` 时绑定到 **SavedStateRegistry** 中，从而在我们 `Activity` 异常重建时做到状态的 **恢复** 与 **绑定 ** (通过重写 `onSavexx()` 与 `onCreate()` 方法监听)。

关于这部分内容，我们下面的源码解析部分也会再聊到，这里我们只需要知道是这么回事即可。

## ViewModel源码解析

本章节，我们将从 **ViewModelProvider()** 开始，理清 `ViewModel` 的 **创建** 与 **销毁** 流程，从而理解其背后的 [魔法]。

不过 ViewModel 的源码其实并不是很复杂，所以别担心😉。

**仔细想想，要解析ViewModel的源码，应该从哪里入手呢？**

```kotlin
ViewModelProvider(this).get(MainViewModel::class.java)
```

最简单的方式还是初始化这里，所以我们直接从 **ViewModelProvider()** 初始化开始-> 

### ViewModelProvider(this)

```kotlin
public constructor(owner: ViewModelStoreOwner)
	: this(owner.viewModelStore, defaultFactory(owner), defaultCreationExtras(owner))
```

相应的，这里开始，我们就涉及到了三个方面，即 **viewModelStore** 、 **Factory**、 **Exras** 。所以接下来我们就顺藤摸瓜，分别看看这三处的实现细节。

#### owner.viewModelStore

![viewmodelprovider-owner](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/viewmodelprovider-owner.viewModelStore.webp)

**ViewModelStoreOwner** 顾名思义，用于保存 `ViewModelStore` 对象。

而 `ViewModelStore` 是负责维护我们 `ViewModel` 实例的具体类，内部有一个 **map** 的合集，用于保存我们创建的所有 `ViewModel` ，并对外提供了 `clear()` 方法，以 **便于非配置变更时清除缓存** 。

---

#### defaultFactory(owner)

![viewmodelprovder-defaultFactory](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/viewmodelprovder-defaultFactory.webp)

该方法用于初始化 `ViewModel` 默认的创造工厂🏭 。默认有两个实现，前者是 **HasDefaultViewModelProviderFactory** ，也是我们 `Fragment` 或者 `ComponentActivity` 都默认实现的接口，而后者是是指全局  **NewInstanceFactory** 。

两者的不同点在于，后者只能创建 **空构造函数** 的 `ViewModel` ，而前者没有这个限制。

**示例源码：**

*HasDefaultViewModelProviderFactory 在 ComponentActivity 中的实现如下:*

![componentAct-HasDefaultViewModelProviderFactory](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/componentAct-HasDefaultViewModelProviderFactory.webp)

---

#### defaultCreationExtras(owner)

用于辅助 `ViewModel` 初始化时需要传入的参数，具体源码如下：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.7jocif8u6a80.webp)

如上所示，默认有两个实现，前者是 **HasDefaultViewModelProviderFactory** ,也就是我们 `ComponentActivity` 实现的接口,具体的实现如下：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.64prl0vy1hk0.webp)

默认会帮我们注入 `application` 以及 `intent` 等，注意这里还默认使用了 **getIntent().getExtras()** 作为 `ViewModel` 的 **默认状态** ，如果我们 `ViewModel` 构造函数中有 `SavedStateHandle` 的话。

> 更多关于 **CreationExtras** 可以了解这篇 [创建 ViewModel 的新方式，CreationExtras 了解一下？](https://blog.csdn.net/vitaviva/article/details/123321254)

---

### get(ViewModel::xx)

从缓存中获取现有的 `ViewModel` 或者 **反射创建** 新的 `ViewModel`。

**示例源码如下：**

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.3tdkg5amxe00.webp)

当我们使用 **get()** 方法获取具体的 `ViewModel` 对象时，内部会先利用 **当前包名+ViewModel类名** 作为 `key` ，然后从 `viewModelStore` 中取。如果当前已创建，则直接使用；反之则调用我们的 ViewModel工厂 **create()** 方法创建新的 `ViewModel`。 创建完成后，并将其保存到 `ViewModelStore` 中。

---

#### create(modelClass,extras)

具体的创造逻辑里，这里的 **factory** 正是我们在 `ViewModelProvider` 初始化时，默认构造函数 **defaultFactory()** 方法中生成的**SavedStateViewModelFactory** ，所以我们直接去看这个工厂类即可。

具体源码如下：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.jukiegc6htc.webp)

---

####  create(key,modelClass)

兼容旧的版本以及用户操作行为。

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.2tq389z49700.webp)

相应的，这里我们还需要再提一下，**LegacySavedStateHandleController.create()** 方法：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.25rg2rlji0g0.webp)

当我们调用创建 `ViewModel` 时，内部会调用具体的 `ViewModel` 工厂去创建，如果当前 `ViewModel` 已创建，则直接返回，否则调用其 **create()** 方法创建新的 `ViewModel` 。在具体的创建方法中，需要判断当前构造函数是不是带 `application` 或者 `SaveStateHandle` ，从而调用合适的 `newInstance()` 方法，最后再将创建好的 `ViewModel` 添加到 `ViewModelStore` 的 **缓存** 中。



---

### 销毁流程

在初始化 `ViewModelProvider` 时，还记得我们需要传递的 `ViewModelStoreOwner` 吗？

而这个接口正是被我们的 ComponentActivity 或者 Fragment 各自实现，相应的 `ViewModelStore` 也是存在于我们的 ComponentActivity 中，所以我们直接去看示例代码即可：

**以ComponentActivity为例，具体的源码如下：**

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.38r5dnhdfeq0.webp)

如上所示：在初始化Activity时，内部会使用 `lifecycle` 添加一个生命周期观察者，并监听 **onDestory()** 通知(Act销毁)，如果当前销毁的原因非配置更改导致，则调用 **ViewModeltore.clear()** ，即清空我们的ViewModel缓存列表，从而这也是为什么 `ViewModel` 不支持非配置更改的实例保存。

**你可能会惊讶，那还怎么借助SavedStateHandle保存状态，viewModel已经被清空了啊🤔?**

如果你记得 `Activity` 传统处理状态的方式，此时也就能理解为什么了？因为源头都是一个地方，而 **SavedStateHandle** 仅仅只是一个更简便的封装而已。不过关于这个问题具体解析，我们将在下面继续进行探讨，从而理解 **SavedStateHandle** 的完整流程。

## SavedStateHandle流程解析

关于 `SavedStateHandle` 的使用方法我们在上面已经叙述过了，其相关的 api 使用源码也不是我们所关注的重点，因为并不复杂，而我们主要要探讨的是其整个流程。

要摸清 `SavedStateHandle` 的流程，无非就两个方向，即 **从何而来** ，又 **在哪里进行使用 ** 🤔。

在上面探索 `ViewModel` 创建流程时，我们发现，在 get(ViewModel:xx) 方法内部,最终的 **create()** 方法里，存在两个分支:

> 1. 存在附加参数extras(viewModel2.5.0新增);
> 2. 不存在附加参数extras(兼容历史版本或者用户自定义的行为);

相应的，如果 `ViewModel` 的构造函数中存在 **SavedStateHandle** ，则各自的流程如下所示：

- CreationExtras.createSavedStateHandle()
- LegacySavedStateHandleController.create(xx).handle

前者使用了 **CreationExtras** 的扩展函数 `createSavedStateHandle()` ：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.jy0kq8fmqh8.webp)

而后者使用了 **LegacySavedStateHandleController** 控制器去创建：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.6kf1qask0hw0.webp)

**总结：**

上述流程中，两者大致是一样的，都需要先调用 `consumeRestoredStateForKey(key)` 拿到要还原的 **Bundle** , 再调用 `SavedStateHandle.createHandle()` 去创建 `SavedStateHandle` 。

那 **SavedStateRegistry** 又是什么呢？

> 我们的插入点也就在于此开始。

我们暂时先不关注如何还原状态，而是先搞清楚 `SavedStateRegistry` 是什么，它又是从哪来而传递来的。然后再来看 状态如何被还原，以及 `SavedStateHandle` 的创建流程，最后再搞清与 `SavedStateRegistry` 又是如何进行关联。

---

### SavedStateRegistry

其是一个用于保存状态的注册表，往往由 **SavedStateRegistryOwner** 接口所提供实现，从而以便与拥有生命周期的组件相关联。

比如我们常用的 `ComponentActivity` 或者 `Fragment` 默认都实现了该接口。

**源码如下所示：**

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.1czwf8l4ksf4.webp)

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.1w9z1jjewli8.webp)

分析上面的代码不难发现，`SavedStateRegistry` 本身提供了状态 **还原** 与 **保存** 的具体能力，并使用一个 **map** 保存当前所有的状态提供者，具体的状态提供者由 **SavedStateProvider** 接口实现。

---

#### SavedStateRegistryOwner

相当于是拥有 `SavedStateRegistry` 的具体类，因为本身继承了 `LifecycleOwner` 接口，故其也具备 **生命感知** 能力，如下所示：

```kotlin
interface SavedStateRegistryOwner : LifecycleOwner {
    val savedStateRegistry: SavedStateRegistry
}
```

以 `ComponentActivity` 为例，我们会发现，`ComponentActivity` 默认实现 **SavedStateRegistryOwner** 接口。即 `SavedStateRegistry` 的创造以及状态的保存，肯定也是 **经过我们Activity转发处理**(不然它自己怎么处理呢😅)。

而在上面探索 **ViewModel** 初始化时，我们了解到，`ComponentActivity` 默认实现了 `HasDefaultViewModelProviderFactory` 接口，用于**创建ViewModel工厂** 。相应的，其接口方法 `getDefaultViewModelProviderFactory()` 默认返回的是 `SavedStateViewModelFactory` ,即支持状态保存的ViewModel工厂。而该工厂构造函数中正是需要接受一个 `SavedStateRegistry` 变量，也正是我们 `ComponentActivity` 中默认保存的实例，所以也不难猜测 **ViewModel工厂** 是如何与 `SavedStateRegistry` 如何关联的。

以 `ComponentActivity` 的实现为例，**源码如下：**

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.2lgj4tmptec0.webp)

`ComponentActivity` 初始化时，会创建一个 用于保存状态注册表的控制器 `SavedStateRegistryController` 对象，见面知意，不难猜出，其是用于控制 `SavedStateRegistry` 的具体类。并且该控制器对象会在 **onCreate()** 中调用 **performRestore()** 还原状态，并在**onSaveInstanceState()** 中去保存状态，此时也就解释了为什么 `SavedStateRegistry` 能做到状态保存。

相应的，我们还是要再去看看 **SavedStateRegistryController** ，以便更好的理解。

---



#### SavedStateRegistryController

用于控制 `SavedStateRegistry` ,对外提供了 **初始化** ，状态 **还原**、**保存** 等方法，如下所示：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.1wyal1vhaaow.webp)

简而言之，其主要用于辅助 **SavedStateRegistry** 进行状态保存与还原。

#### 小结

我们再回顾一下上面的步骤，在只关心 **SavedStateHandle** 如何被创建这样一个大背景下，我们大致可以梳理出这样的流程：

因为我们的 ComponentActivity 或者 Fragment 默认已经实现了 `SavedStateRegistryOwner` 接口，而且默认是由 `SavedStateRegistryController` 作为 `SavedStateRegistry` 的具体控制，因此具体的状态保存与还原都由该控制器去操作。

当我们的 `Activity` 因为异常生命周期重建时，此时会回调 **onSaveInstanceState()** 去保存状态，此时 `SavedStateRegistryController` 就会调用 **performSave()** 去保存当前状态(即将我们ViewModel的状态保存到bundle里)，然后在 Activity 重建时，在 **onCreate()** 方法里进行还原(即从bundle里取出我们保存的状态)。

当我们创建 `ViewModel` 时，默认使用的 `ViewModel` 工厂是支持保存状态的 `SavedStateViewModelFactory` 。在初始化该工厂时，需要显式传递 `SavedStateRegistryOwner` 接口对象到该工厂中，而该工厂的构造函数内，会将 `SavedStateRegistry` 自行保存起来。

最后，如果要创建的 `ViewModel` 需要保存状态(**即构造函数中存在SavedStateHadnle**)，则使用保存的 `SavedStateRegistry` 变量去获取我们将要还原的状态，然后再调用 `SavedStateHandle.createHandle()` 去创建具体的 `SavedStateHadnle`。

由此结合 `ViewModel` 创建的流程，我们可以总结 `SavedStateRegistry` 的传递流程伪代码如下：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.r0qq46076tc.webp)

---



### SavedStateHandle如何创建

在上面，我们聊完了 `SavedStateRegistry` 是如何被创建以及被传递给我们的 **ViewModel工厂** ，而这一小节，我们将要聊聊 `SavedStateHandle` 如何被创建，以及状态是如何被还原的。

我们知道，当创建 `SavedStateHandle` 前，需要先获取已保存的状态，也即 `consumeRestoredStateForKey()` 方法，所以我们本章节的插入点也就是从这里开始。

而与 `consumeRestoredStateForKey()` 关联的类有两个, **SavedStateHandlesProvider** 与 **SavedStateRegistry** 。

> 前者是 `viewModel`(2.5.0) 新提供的 **创建SavedStateHandle** 的方式，后者则是用于 **适配** 2.5.0 之前的方式。

以 **SavedStateHandlesProvider** 为例，源码如下：

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.7glywcc896s0.webp)

当我们调用 `consumeRestoredStateForKey()` 获取具体状态时，内部先会调用 `performRestore()` 从 **SavedStateRegistry** 获取我们保存的状态集，然后将其保存到 `provider` 中。再从这个总的 **状态bundle** 中获取我们当前 `viewModel` 所对应的状态。

相应的，我们再去看看 **SavedStateHandle.createHandle()** 方法，即 `SavedStateHandle` 最终被怎么创建出来。

**源码如下：**

![petterp-image](https://cdn.staticaly.com/gh/Petterpx/ImageRespoisty@main/img/petterp-image.747mr5tm6tg0.webp)

上述的逻辑也比较简单，具体如源码中所示，当我们创建 **SavedStateHandle** 时，需要先从 **SavedStateRegistry** 获取我们的状态Bundle,然后再调用 `createHandle()` 方法创建具体的 **SavedStateHandle**。并在其 `createHandle()` 内将我们传入的 **bundle** 转为 **Map** 形式，从而传入 **SavedStateHandle** 的构造函数中用于初始化。

### 总结

在这一章节，我们主要探讨的是 `SavedStateHandle` 的创建流程，以 `ComponentActivity` 为例：

我们知道 Android 中关于状态的保存与还原，官方建议使用 **onSaveInstanceState()** 与 **onRestoreInstanceState()** ，但随着JetPack组件库的完善，官方在这两个方法的基础上新增了 `SavedState` ,目的是简化状态保存的成本。从原理上，其创建了一个 状态保存的的注册表 `SavedStateRegistry` ，内部缓存着具体的 **状态提供者合集**(key为string,value为SavedStateProvider)。

当我们 Activity 因为配置更改或者不可控原因需要重建时，系统此时会主动调用 **onSaveInstanceState()** 方法，从而触发调用 `savedStateRegistry.performSave()` 去保存状态。该方法内部会创建一个新的 **Bundle** 对象，用于保存所有状态,然后再调用所有缓存的状态提供者(SavedStateProvider)的 `saveState()` 方法，从而将所有需要需要保存的状态以 **key-value** 的方式存到 **Bundle** 中去。最后再将这个整体的 **bundle** 存入 `onSaveInstanceState()` 方法参数提供的 **bundle** 中。

当我们的 Activity 重建完成后，在 `onCreate()` 方法中，再使用 `SavedStateRegistry` 还原我们自己保存的状态 **restoredState**。

最后当我们创建 `ViewModel` 时，因为我们的 **ViewModel工厂**(SavedStateViewModelFactory) 持有了 `SavedStateRegistry` ，也即持有着我们要还原的状态(如果有)。在创建具体的 `ViewModel` 时，如果我们要创建的 `ViewModel` 构造函数中存在 `SavedStateHandle` 参数，则该 `ViewModel` 支持保存状态，所以需要先去使用 `SavedStateRegistry` 获取我们保存的状态，最后再调用 **SavedStateHandle.create()** 去创建具体 `SaveStateHandle` ，从而创建出支持保存状态 `ViewModel` 。

## 结语

在本篇中，我们从 `ViewModel` 的背景开始，再到 `ViewModel` 与 `SavedStateHandle` 的使用方式，最后又从源码层级分析了两者的具体流程，从而较完整的解析了 `ViewModel` 的底层实现与 `SavedStateHandle` 的整体创建流程。

至于更加详细的使用方式，这也非本篇要深入探索的细节，具体可参照其他同学的教程即可。至此，关于 `ViewModel` 设计思想 以及 **状态保存原理** ，相信读过本篇的你也将不会有所疑问。

## 参阅

- [Android开发者-ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [创建 ViewModel 的新方式，CreationExtras 了解一下？](https://blog.csdn.net/vitaviva/article/details/123321254)

## 更多

这是 `JetPack` 组件系列 - **ViewModel** 篇，如果你觉得这个系列写的还不错，也可以看看其他篇：

> - [由浅入深，详解 Lifecycle 的那些事](https://juejin.cn/post/7168868230977552421)
> - [由浅入深，详解 LiveData 的那些事](https://juejin.cn/post/7173494700081414181)

## 关于我

## 

## 

我是 *Petterp* ,一个 Android工程师 ，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！

