# 求知 | Android资源加载的那些事-小试牛刀

## 引言

聊到到 Android 的 **资源加载** ，每一个开发同学都会非常熟悉，毕竟 `getText()` 等, 我们实在用了太多。

那如果此时问你，**你知道 它们到底是怎么被加载的，内部会有什么处理吗? 为什么同一个drawable界面更改了透明度，其他界面也会生效?** 

如果你对上述问题依然存疑，那本文可能会对你有所帮助。

介于此，本篇将由浅入深，从源头理清 `Resource.getx()` 的那些事，从而为理解 Android资源加载 迈出第一步。故此名: **小试牛刀**。

> 本篇定位中等📌，主要通过伪源码分析的形式，从而探索应用层 Resource.getx 的实现细节。
>

## Resource是什么？

`Resource`，在 Android 中，指的是我们开发中使用到的资源，例如 `drawable`、`String`、`anim`、`color` 等。其会在开发阶段生成相应的R类以及对应的 **资源ID** ，以便开发者在使用时通过传递 **资源Id** ,从而获取相应类型的资源文件。

比如我们在 `Activity`,`Fragment` 中经常使用的 **getString()** , **getDrawable()** ,内部也都是调用的 `resource.getxx` 实现。

**常见也有 ContextCompat.getDrawable() ,那它与直接调用resource.getDrawable()有什么区别？** 

见名知意，其主要是作为兼容使用，目的是解决不同版本之间的差异。

## 基础概念

**TypedValue**

用于保存数据的动态容器，主要用于配合 `Resource` 保存资源。

具体而言，当我们获取资源时，底层会调用相应的原生方法将读取到的资源信息写入其中，以便后续的判断与使用；

**AssetsManager**

资源管理器，用于读取打包到 `Apk` 内部的资源文件。

具体而言，当我们调用 `getxxx` 时，其最终会去调用相应的原生方法获取资源信息并写入 `TypedValue` ；

**ResourcesImpl**

`Resource` 的具体实现类，我们调用的相关 `getxxx` 方法，最终都是其作为具体实现，内部最终会调用 `AssetsManager` 进行加载资源，并且会处理与之关联的所有缓存。

## getText

`getText(R.string.xx)`

用于从资源文件中获取文本，具体源码如下：

![image-20221009221309098](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210092213188.png)

> 从源码中看，我们调用的 `getText()` 最终实际调用了 `ResourcesImpl` , 内部会使用 `AssetsManager` 去从底层获取相应的文本资源，并将其保存到 `TypedValue` 中。如果此次获取的文本资源是字符串类型，则直接从字符串常量池中去取，否则将取到的文本资源转为字符串后返回。

## getDrawable

`getDrawable(R.drawable.xxx)`

用于从资源文件中获取可绘制对象,具体伪源码如下：

![image-20221009223642840](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210092236877.png)

> 当我们调用 `getDrawable()` 时,内部先会通过 `getValueForDensity()` 获取当前密度下相应的资源文件，并将其写入到 `TypeValue` 中；如果不存在资源文件，则直接抛出异常。然后通过 `ResourcesImpl.loadDrawable` 去加载 `Drawable`。

---





继续沿着刚才的源码，我们去看看 `loadDrawable` 内部到底做了什么，伪代码如下：

![image-20221008223438146](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210082234173.png)

这个方法流程较长，我们将其分为下面几个步骤：

1. 判断当前要加载的 `drawable` 是否具有缓存；

2. 判断当前 `drawable` 是否为颜色drawable；

3. 如果当前没有加载 `drawable` &&当前drawable **已缓存** ,直接返回该drawable；

4. 从当前缓存中取出当前 `drawable` 对应的 **状态与数据参数(如果存在缓存)** ；

5. 创建新的 `drawable` 。如果当前存在缓存，则利用缓存的状态(Drawable.ConstantState) 构建 `Drawable`，否则如果是颜色drawable，则直接创建；否则调用 从xml或者资源中加载drawable,具体伪代码如下图：

   > ![image-20221011204058689](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210112040723.png)

6. 处理构建的drawable **主题与参数** ；

7. 如果当前drawable **没有缓存** ，则将添加到缓存中。

------



### 总结

当我们调用 `getDrawable()` 时,内部会先判断当前资源是否存在，如果不存在则直接抛出异常；接着调用 `ResourcesImpl.loadDrawable` 去加载具体的 `drawable` ,内部会根据要加载的 `drawable` 的 **类型**、是否是Color,以及是否存在缓存综合获取，如果存在当前屏幕密度的drawable,则使用缓存，否则重新加载。然后根据要加载的 `drawable` **文件后缀** 决定是 `colorDrawable` 还是 `BitMapDrawable` ，或者是其他类型的Drawable，最后将加载完成的 `Drawable` 的 **状态与配置参数(ConstantState)** 加入到 **缓存** 中。

----



### Tips

知道了 `Drawable` 会被缓存的知识点，此时就不难解释为什么开发中会遇到同一个 `Drawable` 更改了透明度，其他界面用到这个 `Drawable` 的地方也会受到了影响。

如下示例:

![Kapture 2022-10-08 at 23.31.52](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210082332367.gif)

解决办法就是，在 `drawable` 更改透明度时，调用 `mutate()` 即可,原理上也很简单，重新new了一个状态：

```kotlin
background.mutate().alpha = 100
```

**例如：**

![image-20221011204033037](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210112040074.png)

## getColor

**getColor(R.color.xxx)**

用于获取相应 `资源id` 关联的颜色,具体的源码如下：

![image-20221009230046256](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210092300310.png)

> 当我们调用 `getColor()` 时,内部先会通过 `getValue()` 获取相应的 `color` 资源，并将其保存到 `TypeValue` 中；如果不存在资源文件，则直接抛出异常。然后通过 `ResourcesImpl.loadColorStateList()` 去加载,最后返回颜色状态列表的 **默认显示颜色**。

----



我们继续向下看： **loadColorStateList()**

![image-20221011203842633](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210112038667.png)

> 当调用 `loadColorStateList` 加载颜色状态合集时，内部有两个分支：
>
> - 如果当前要获取的颜色类型是 **“#xxx”** ,则先从预加载数组中取，如果此时没有加载，则创新的 `ColorStateList` ,并将其存到预加载数组中；
> - 如果当前要获取的颜色类型是引用类型，则意味着当前可能要从xml中去取。内部先从缓存数组中去，如果不存在则再去预加载数组中取，如果依然不存在，则调用 `loadComplexColorForCookie()` 重新初始化。当加载完成后，如果此时正在预加载，将其添加到预加载数组中，否则将其添加到缓存里。

-----



接着上面的末梢，我们最后再去看一下 `loadComplexColorForCookie()` ,也即**一个全新的color到底是如何从xml中拿到**：

![image-20221011203812474](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202210112038594.png)

> 该方法里，先判断资源文件的后缀名，如果非 `.xml` 类型，则该资源无法读取，直接抛出异常；否则先调用 `loadXmlResourceParser() `拿到该资源文件的 **xml解析器** ，再由解析器的 `name` 判断具体的资源类型，从而初始化具体的颜色类。

### 总结

当我们调用 `getColor()` 获取某个颜色资源时，内部会先通过 `AssetsManager` 加载该资源，并将其保存到 `TypedValue` 中，如果没有读到，则抛出异常；否则调用 `ResoucesImpl.loadColorStateList()` 获取颜色资源，如果该资源在缓存中存在，则直接取出并返回新的实例，否则根据当前要加载的类型，如果是 **“#xxx”** ，则直接初始化并添加到缓存，否则判断 `TypedValue` 中保存的资源信息 **后缀** 是否为 `xml` ,如果不是则直接抛出异常，证明此时非 **.xml** 文件，文件无法读取，否则通过 `AssetManager` 获取该资源对应的 **xml解析器** ，并判断解析器的名字，从而决定创建 `GradientColor` 还是 `ColorStateList`，然后将结果缓存到 `ResourcesImpl` 中并返回。

## 结语

关于 `Resource.getx()` 相关的底层实现到这里就分析结束了。本篇中，我们以 **Kotlin+[裁枝剪叶]** 的方式，提供一个较清晰的脉络，以供更好的读懂应用层源码设计，关于更细节的原生实现，并不是本篇所关注的。所谓一眼入森，而不在林，正是如此。

现在让我们反推上去：

原来我们每次调用 `getDrawable()` 时，内部都是做了缓存处理(缓存了`ConstantState`),原来我们获取的 `drawable`,无非就三种大的类型:

- **.xml** 结尾的，`ColorDrawable` 或者 `非ColorDrawable`;
- **非.xml** 结尾的，即为 `BitmapDrawable`。

那他们又是怎么判断得出的呢？通过 `AssetManager` 获取，将其保存到 `TypedValue` 中，使用时通过判断 **资源文件名后缀** 而定。又因为`drawable` 存在 **缓存状态复用** ，所以又会导致 **一处更新，处处同步** 问题。原来 `getColor()` 内部同样做了缓存处理等。

> 至此，关于` Android-Resource` 的求知篇正式开始，下一篇我将同大家分析 `Resource` 的初始化时机以及与 `Resource.system()` 的区别。

## 关于我

我是 Petterp ,一个 Android工程师 ，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！
