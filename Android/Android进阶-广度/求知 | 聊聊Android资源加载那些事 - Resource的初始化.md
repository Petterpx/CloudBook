# 求知 | 聊聊Android资源加载那些事 - Resource的初始化

Hi,你好 :)

## 引言

在上一篇，[求知 | 聊聊Android资源加载的那些事 - 小试牛刀](https://juejin.cn/post/7153266829471776798) 中，我们通过探讨 `Resource.getx()` ,从而解释了相关方法的背后实现, 明白了那些我们日常调用方法的背后实现。

那么，不知道你有没有好奇 `context.resources` 与 `Resource.getSystem()` 有什么不同呢？前者又是在什么时候被初始化的呢？

如果你对上述问题依然存疑，或者你想在复杂中找到一个较清晰的脉络，那本文可能会对你有所帮助。

介于此，本篇将与你一同探讨关于 `Resources` 初始化的那些事，从而建立起应用层资源加载的整体脉络。故名 渐入佳境；

> 本篇定位中等，主要通过伪源码的方式探索 `Resources` 的初始化过程📌
>
> **sdk版本 :31**

## 导航

学完本篇，你将明白以下内容：

- `Resource(Activity)`、`Resource(App)` 初始化流程
- `Context.resource` 与 `Resource.getSystem()` 的不同之处

## 基础概念

开始本篇前，我们先了解一些必备的基础概念：

- **ActivityResource**

  用于持有 `Activity` 或者 `WindowContext` 相关联的 `Resources` ；

- **ActivityResources**

  用于管理 `Acitivty` 的 `config` 和其所有 `ActivityResource` ,以及当前正在显示的屏幕id;

- ResourceManager

  用于管理 `App` 所有的 `resources`，内部有一个 `mActivityResourceReferences` map保存着所有 `activity` 或者 `windowsToken` 对应的 `Resources` 对象。

## Resource(Activity)

在 `Activity` 中调用 **getX()** 相关方法时,点进源码不难发现，内部都是调用的 **getResource().x** ，而 `getResource()` 又是来自 `Context` ，所以一切的源头也即从这里开始。

了解 `context` 的小伙伴应该有印象， `context` 作为一个顶级抽象类，无论是 `Activity` 还是 `Application` 都是其的子类， `Context` 的实现类又是 `ContextImpl`，所以当我们要找 `Activity` 中 `resource` 在哪里被初始化时🧐，也即是在找：

-> `ContextImpl.resource` 在哪里被初始化? ➡️

顺藤摸瓜，我们去看看 `ContextImpl.createActivityContext()`。

> 该方法的调用时机是在构建我们 `Activity` 之前调用,目的是用于创建 `context` 实例。

### 流程分析

**具体如下：**

`ContextImpl - createActivityContext`

```java
ContextImpl createActivityContext(
						ActivityThread mainThread,
            LoadedApk packageInfo, 
  					ActivityInfo activityInfo, 
  					IBinder activityToken, int displayId,
            Configuration overrideConfiguration) {
  	
  	// 获取当前的分辨率文件
    String[] splitDirs = packageInfo.getSplitResDirs();
  	// 获取classLoader
    ClassLoader classLoader = packageInfo.getClassLoader();
		...
 		// 初始化context
  	ContextImpl context = new ContextImpl();
    context.mContextType = CONTEXT_TYPE_ACTIVITY;
  	// 获取资源管理器单例
    final ResourcesManager resourcesManager = ResourcesManager.getInstance();
		...
		// 根据此Activity的配置 设置资源对象
    context.setResources(resourcesManager.createBaseTokenResources());
    return context;
}
```

上述总结如下：

> 内部会获取当前的 **分辨率** 、`classLoader` 等配置信息，并调用 `ResourcesManager.getInstance()` 从而获取 `ResourcesManager` 的单例对象，然后使用其的 `createBaseTokenResources()` 去创建最终的 `Resources` 。
>
> 接着 将resource对象保存到 `context` 中,即赋值给 `ContextImpl.mResources` 。
>
> ps: 如果 **sdk>=26** ,还会做 `CompatResources` 的判断。

了解了上述流程，我们接着去看 `resourcesManager.createBaseTokenResources`() 。

---

`ResourceManager.createBaseTokenResources()`

```java
public Resources createBaseTokenResources(
  			IBinder token,
				...  
){
        final ResourcesKey key = new ResourcesKey(resDir,...,displayId,...);
        classLoader = classLoader != null ? classLoader : ClassLoader.getSystemClassLoader();
  			//📌1. 创建对应的 activityResources 对象(如果没创建) ➡️
        synchronized (mLock) {
            getOrCreateActivityResourcesStructLocked(token);
        }
				//📌2. 更新该token对应的resources  ➡️
        updateResourcesForActivity(token, overrideConfig, displayId);
				
  			//📌3. 查找当前活动对应的资源对象,存在则复用
        synchronized (mLock) {
            Resources resources = findResourcesForActivityLocked(token, key,classLoader);
            if (resources != null) {
                return resources;
            }
        }

        //📌4. 如果还不存在资源对象,则创建资源对象
        return createResourcesForActivity(.token,key...);
}
```

上述总结如下：

> 该方法用于创建当前 `activity` 相对应的 `resources` ,内部会经历如下步骤：
>
> 1. 先查找或创建当前 **token(activity)** 所对应的 `resources` ；
>
>      Yes -> 什么都不做;
>
>      No  -> 创建一个 `ActivityResources` ，并将其添加到 `mActivityResourceReferences` map中;
>
> 2. 接着再去更新该 `activity` 对应 `resources`(内部会再次执行第一步);
>
> 3. 再次查找当前的 `resources` ,如果找到，则直接返回；
>
> 4. 如果找不到，则重新创建一个 `resources`(内部又会再执行第一步);

具体的步骤如下所示：

---

-1. **getOrCreateActivityResourcesStructLocked()**

ResourcesManager.getOrCreateActivityResourcesStructLocked()

```java
private ActivityResources getOrCreateActivityResourcesStructLocked(
        IBinder activityToken) {
  	// 先从map获取
    ActivityResources activityResources = mActivityResourceReferences.get(activityToken);
  	// 不存在,则创建新的，并以token为key保存到map中，并返回新创建的ActivityResources
    if (activityResources == null) {
        activityResources = new ActivityResources();
        mActivityResourceReferences.put(activityToken, activityResources);
    }
    return activityResources;
}
```

正如题名，获取或创建 `ActivityResources` 。如果存在则返回，否则创建并保存到 **ResourcesManager.mActivityResourceReferences**中。



-2. **updateResourcesForActivity()**

ResourcesManager.updateResourcesForActivity()

```java
public void updateResourcesForActivity(...) {
  		 // 获取当前activity对应的resources集
       final ActivityResources activityResources =
               getOrCreateActivityResourcesStructLocked(activityToken);
  		 // 📌 1. 如果当前的配置信息与传入的一致,则直接退出
       if (Objects.equals(activityResources.overrideConfig, overrideConfig)...) {
           return;
       }
  
  		 // 📌 2.用activityResources 创建旧的配置对象
       final Configuration oldConfig = new Configuration(activityResources.overrideConfig);
       if (overrideConfig != null) {
           activityResources.overrideConfig.setTo(overrideConfig);
       } else {
           activityResources.overrideConfig.unset();
       }
			 ...
       
       // 📌 3.更新当前actiivty关联的所有resources的基础实现
       final int refCount = activityResources.activityResources.size();
			 for (int i = 0; i < refCount; i++) {
         	 ActivityResource activityResource = activityResources.activityResources.get(i);
           Resources resources = activityResource.resources.get();
           // 使用新的配置信息覆盖当前Resource的配置,内部实现差异更新(非直接覆盖),然后生成新的key(如果有更新)
         	 ResourcesKey newKey = rebaseActivityOverrideConfig(activityResource,
                            overrideConfig, displayId);
           if (newKey == null) continue;
         	 // 查找或创建resourcesImpl
      		 ResourcesImpl resourcesImpl = findOrCreateResourcesImplForKeyLocked(newKey);
         	 // 如果impl已改变,则重新设置
         	 if(resourcesImpl != resources.getImpl())
           		resources.setImpl(resourcesImpl);
       }
}
```

**流程如下：**

内部会先获取当前 `activity` 对应的 `resources`(如果不存在,则创建)，**如果当前传入的配置与之前一致，则直接返回**。

否则先使用当前 `activity` 对应的配置 创建一个 **[旧]配置对象**，接着去更新该 `activity` 所有的 `resources` 具体实现类`impl`。每次更新时会先与先前的配置进行差异更新并返回新的 `ReourcesKey` ,并使用这个 `key` 获取其对应的 `impl` (如果没有则创建),获取到的 `resource` 实现类 `impl` 如果与当前的不一致，则更新当前 `resources` 的 `impl`。



**-3. findResourcesForActivityLocked()**

ResourcesManager.findResourcesForActivityLocked()

```java
Resources findResourcesForActivityLocked(IBinder token,
                                         ResourcesKey targetKey,
                                         ClassLoader targetClassLoader) {
  	// 获取或创建当前token对应的resources
    ActivityResources activityResources = 
      getOrCreateActivityResourcesStructLocked(targetActivityToken);
		
  	// 遍历resources列表,获取符合当前要求的
    int size = activityResources.activityResources.size();
    for (int index = 0; index < size; index++) {
        ActivityResource activityResource = activityResources.activityResources.get(index);
        Resources resources = activityResource.resources.get();
        ResourcesKey key = resources == null ? null : findKeyForResourceImplLocked(
                resources.getImpl());
				// 如果符合要求,则返回当前resources,否则返回null
        if (key != null
                && Objects.equals(resources.getClassLoader(), targetClassLoader)
                && Objects.equals(key, targetKey)) {
            return resources;
        }
    }
    return null;
}
```

上述流程如下：

当通过 `findResourcesForActivityLocked()` 获取指定的 `resources` 时，内部会先获取当前 `token` 对应的 `activityResources` ，从而拿到其所有的 `resources` ；然后遍历所有 `resources` ,如果某个 `resouces` 对应的 **key(ResourcesKey)** 与当前查找的一致并且符合其他规则，则直接返回，否则无符合条件时返回null。



**–4.createResourcesForActivity()**

```java
private Resources createResourcesForActivity(IBinder activityToken,
       ResourcesKey key,Configuration initialOverrideConfig,
       Integer overrideDisplayId,ClassLoader classLoader,
       ApkAssetsSupplier apkSupplier) {
  			// 查找或创建相应的key
        ResourcesImpl resourcesImpl =
          findOrCreateResourcesImplForKeyLocked(key, apkSupplier);
        if (resourcesImpl == null) return null;
  			// 创建新的resources
        return createResourcesForActivityLocked(activityToken, initialOverrideConfig,
                overrideDisplayId, classLoader, resourcesImpl, key.mCompatInfo);
    }
}

private Resources createResourcesForActivityLocked() {
  			// 获取ActivityResources(不存在则创建新的)
        ActivityResources activityResources = 
          	getOrCreateActivityResourcesStructLocked(activityToken);
  			...
        // 创建新的Resources,这里还需要判断是否是Compat(sdk>=21)
        Resources resources = compatInfo.needsCompatResources() 
          ? new CompatResources(classLoader) : new Resources(classLoader);
        resources.setImpl(impl);
				...
        ActivityResource activityResource = new ActivityResource();
        activityResource.resources = new WeakReference<>(resources,
                activityResources.activityResourcesQueue);
        ...
        activityResources.activityResources.add(activityResource);
        return resources;
    }
```

上述流程如下：

在创建 `Resources` 时，内部会先使用 `key` 查找相应的 `ResourcesImpl` ,如果没找到，则直接返回null,否则调用 `createResourcesForActivityLocked()` 创建新的Resources.

### 总结

当我们在 `Activity`、`Fragment` 中调用 `getX()` 相关方法时,由于 `context` 只是一个代理，提供了获取 `Resources` 的 `getx()` 方法，具体实现在 `ContextImpl`。所以在我们的 `Activity` 被创建之前，会先创建 `contextImpl`,从而调用 `createActivityContext()` 这个方法内部完成了对 `resources` 的初始化。内部会先拿到 `ResourcesManager`(用于管理我们所有resources),从而调用其的`createBaseTokenResources()` 去创建所需要的 `resources` ,然后将其赋值给 `contextImpl`。

在具体的创建过程中分为如下几步：

1. 先从 `ResourcesManager` 缓存 **(mActivityResourceReferences)** 中去找当前 **token(Ibinder)** 所对应的 `ActivityResources`,如果没找到则重新创建一个，并将其添加到 `map` 中；
2. 接着再去更新当前 `token` 所关联 `ActivityResources` 内部(activityResource)所有的resources,如果现有的配置参数与当前要更新的一致，则跳过更新，否则遍历更新所有 `resources`;
3. 再去获取所需要的 `resources` ,如果找到则返回,否则开始创建新的 `resources`；
4. 内部会先去获取 `ResourcesImpl`,如果不存在则会创建一个新的，然后带上所有配置以及 **token** 去创建相应的 `resources` ,内部也同样会执行一遍第一步,然后再创建 `ActivityResource` ,并将其添加到第一步创建的 `activityResouces` 中。

## Resrouces(Application)

我们应该从哪里找入口呢？🧐

既然是 `Application` 级别，那就找找 `Application` 什么时候初始化？而 `Resources` 来自 `Context` ,所以我们要寻找的位置又自然是 `ContextImpl` 了。故此，我们去看看 `ContexntImpl.createSystemContext()` :

> 该方法用于创建 `App` 的第一个上下文对象，即也就是 `AppContext`。

### 流程分析

**ContexntImpl.createSystemContext()**

```kotlin
fun createSystemContext(mainThread:ActivityThread) {
  	// 创建和系统包有关的资源信息
    val packageInfo = LoadedApk(mainThread)
    ...
    val context = ContextImpl(xxx)
  	➡️
    context.setResources(packageInfo.getResources())
    ...
    return context
}
```

如上所示，当创建系统 `Context` 时，会先初始化一个 `LoadedApk` ,用于管理我们系统包相关信息，然后再创建 `ContextImpl` ，然后调用创建好的 `LoadedApk` 的 `getResources()` 方法获取系统资源对象，并将其设置给我们的 `ContextImpl` 。

➡️ **LoadedApk.getResources()**

```java
fun Resources getResources() {
    if (mResources == null) {
        ...
        ➡️➡️
        mResources = ResourcesManager.getInstance().getResources(null, mResDir,
                splitPaths, mLegacyOverlayDirs, mOverlayPaths,
                mApplicationInfo.sharedLibraryFiles, null, null, getCompatibilityInfo(),
                getClassLoader(), null)
    }
    return mResources
}
```

当我们获取 `resources` 时，内部会先判断是否存在，如果不存在，则调用 `ResourcesManager.getResources()` 去获取新的 `resources` 并返回,否则直接返回现有的。相应的，我们再去看看 `ResourcesManager.getResources()` 。

➡️➡️ **ResourcesManager.getResources()**

```kotlin
fun getResources(activityToken:IBinder,...):Resources {
  
     🔺 activityToken != null
     			->  createResourcesForActivity()
           			➡️
  							fun createResourcesForActivity():Resources{
                   val impl = findOrCreateResourcesImplForKeyLocked()
                   return createResourcesForActivityLocked(..impl..)
                }
  
     🔺 activityToken == null
         	->  createResources(key, classLoader, assetsSupplier)
        				➡️
  							fun createResource():Resources{
                   // 查找或创建相应的ResourcesImpl
                   val impl = findOrCreateResourcesImplForKeyLocked()
                   return createResourcesLocked(..impl..)
                }
  
}
```

如上所示,内部会对传入的 `activityToken` 进行判断,如果为不为 null ,则调用 `createResourceForActivity()` 去创建；否则调用 `createResources()` 去创建，具体内部的逻辑和最开始相似，内部会先使用 `key` 查找相应的 `ResourcesImpl` ,如果没找到，则分别调用相关方法再去创建 `Resources` 。

关于 `createResourceLocked()` ,我们再看一眼,如下所示：

> ```kotlin
> fun createResourcesLocked(cls:ClassLoader,
>                        		impl:ResourceImpl,
>                        		compatInfo:CompatibilityInfo): Resources {
>  ...
>  // 创建新的Resources(并判断当前是否为Compat版本,即sdk version>=21)
>  val resources = compatInfo.needsCompatResources()
> 									? CompatResources(classLoader) : Resources(classLoader)
>  resources.setImpl(impl)
>  ...
> 	// 将其添加到缓存list中,便于重复复用
>  mResourceReferences.add(WeakReference(resources, mResourcesReferencesQueue))
>  return resources
> }
> ```
>
> 这个方法内部创建了一个新的 `resources` , 最终将其add到了 `ResourcesManager.mResourceReferences` 这个List中，以便复用。

### 总结

当我们的 `App` 启动后，初始化 `Application` 时，会调用到 `ContexntImpl.createSystemContext()` ,该方法内部同时也会完成对我们`Resources` 的初始化。内部流程如下：

1. 先初始化 `LoadedApk` 对象(其用于管理app的信息)，再调用其的 `getResources()` 方法获取具体的 `Resources`；
2. 在上述方法内部，会先判断当前 `resources` 是否为 **null**。 如果为null,则使用 `ResourcesManager.getResources()` 去获取,因为这是 `application` 的初始化，所以不存在 `activityToken` ,故内部会直接调用 `ResourceManager.createResource()` 方法，内部会创建一个新的 `Resources` 并将其添加到 `mResourceReferences` 缓存中。

## Resources(System)

大家都应该见过这样的代码，比如 `Resources.getSystem().getX()` , 而他内部的实现也非常简单，如下所示：

```kotlin
fun Resources getSystem() {
  			...
        val ret:Resources = mSystem
        if (ret == null) {
            ret = Resources()
            mSystem = ret
        }
        return ret
}

 private Resources() {
				...
        final DisplayMetrics metrics = new DisplayMetrics()
   			// 使用默认设备显示器信息
        metrics.setToDefaults();
        final Configuration config = new Configuration()
     		// 使用默认配置
        config.setToDefaults()
   			🔺 // 这是Resources.getSystem()与context.getSystem()的不同之处
        mResourcesImpl = new ResourcesImpl(AssetManager.getSystem()...)
   			
   			-> fun AssetManager.getSystem(){
            	//FRAMEWORK_APK_PATH = "/system/framework/framework-res.apk"
           	 return createSystemAssetsInZygoteLocked(false, FRAMEWORK_APK_PATH)
       		 }
    }
```

### Tips

当我们使用 `Resources.getSystem()` 时，其实也就是在调用当前 `formwork` 层的资源对象，内部会先判断是否为 **null**,然后进行初始化，初始化的过程中,因为系统框架层的资源，所以实际的资源管理器直接调用了 `AssetManager.getSystem()` ,这个方法内部会使用当前系统框架层的apk作为资源路径。所以我们自然也无法用它去加载我们 `Apk` 内部的资源文件。

## 小问题

在了解了上述流程后，如果你存在以下问题(就是这么倔强🫡)，那么不妨鼓励鼓励自己，[你没掉队]!

1. **为什么要存在 ActivityResources 与 ActivityResource ? 我们一直调用的不都是Resources吗？**

```kotlin
class ActivityResource {
        val Configuration overrideConfig = Configuration()
        var overrideDisplayId : Int
        var resources : WeakReference<Resources>
}

class ActivityResources{
  	    var Configuration overrideConfig = Configuration()
  			var overrideDisplayId : Int
  			val activityResources : ArrayList<ActivityResource>
  			val activityResourcesQueue : ReferenceQueue<Resources>
}
```

首先说说 `ActivityResource` ,见名知意，它是作为 `Resources` 的包装类型出现，内部持有当前要加载的配置，以及真正的 `Resources` 。又因为一个 `Activity` 可能关联多个 `Resources` ,所以 `ActivityResources` 是一个 `activity`(或者`windowsContext`) 的所有 `resources` 合集，内部用一个List维护,而 `ActivityResources` 又被 `ResourcesManager` 缓存着。

2. **Resources.getSystem() 获取应用drawable,为什么会报错？**

   原因也很简单啊,因为 `resources` 相应的 `AssetManager` 对应的资源路径时 `formWork` 啊,你让它获取当前应用资源，它不造啊。🥲

## 结语

最终，让我们反推上去，总体再来回顾一下 `Resources` 初始化的相关:

- 原来我们的 `resources` 都是在 `context` 创建时初始化，而且我们所调用的 `resources` 实际上被 `ActivityResource` 所包装;
- 原来我们的 `Resources` 只是一个代理，最终的调用其实是 `ResourcesImpl` ,并且被 `ResourcesManager` 所缓存。
- 原来每当我们初始化一个 `Activity` ,我们所有的 `resources` 都会被刷新，为什么呢，因为我们的 `resources` 配置可能会改变。
- 原来 `Resource.getSystem()` 无法加载应用资源的原因只是因为 `AssetManager` 对应的资源路径是 `formeWrok` 。

> 本篇中，我们专注于一个概念，即：**resources 到底从何而来**，并且从原理上分析了不同context `resources` 的初始化流程，也明白了他们之间的区别与差异。下一篇我将同大家分析 `ResourcesManager` ,即理解为什么要这么管理。👋

