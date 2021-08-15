# 记一次inline使用不当导致编译期Null指针的排查过程

## 起因

周五的一个下午，我哼着小曲和往常一样合完代码。准备运行试试看，结果build时发现了这样一个异常。

`InlineParameterChecker NullPointerException`

![image-20210815115927974](https://tva1.sinaimg.cn/large/008i3skNly1gthc7q7koqj625s0e8tfl02.jpg)

一般对于这种编译期间的异常，原因往往并不是很容易能快速定位，因为往往都是业务代码出现的问题，如果某次合并更改很多，比如我这一次，重构了底层的某个组件，所以直接当场裂开😑。于是接下来整个任务都变成了如何找到 **错误的** 代码处。

## 先说结论

> 当方法添加了 `inline` 修饰后，即也就是内联之后，如果方法参数是一个函数对象(`lambda`)，那么不可为 `null`。

一般情况下 `IDE` 会主动提示你，如下所示：

![image-20210815121248484](https://tva1.sinaimg.cn/large/008i3skNly1gthclkq28aj61ka084jsn02.jpg)

但是特殊情况下，如下错误示例：

![inline-ide错误示例](https://tva1.sinaimg.cn/large/008i3skNly1gthcmpan46g60me0a2b2h02.gif)

> 某一天，程序员小P 突然发现一段代码，善用Kotlin的他，觉得这里可以使用 `inline` 可以优化，于是下意识就加了一个 `inline` ,但是没有仔细看方法参数，反正IDE没报错，即也就是自己还没意识这个更改带来的严重性。
>
> 但是一旦改完之后，没有 `build` ，那么这就是一个隐藏的坑，严重一点可能会导致你好几个小时找不到原因。



## 如何定位错误代码

如果直接对着代码找，那么可能就需要对比所有相关 `inline` 相关的代码，如果使用之处不多，那么也能很快定位。

但是最关键的问题在于: **我怎么知道那段代码有问题呢，虽然我知道是inline的问题，但是具体是什么呢，我现在不知道啊😢**，所以这种方法暂时也只能放弃。

### google三连

身为一个开发经验小成的老鸟，这肯定难不倒我，直接google三连

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gthd0bsb6vj60w80u0dlf02.jpg" alt="image-20210815122659073" style="zoom:50%;" />

什么玩意都是，我是傻狗。

难道不应该直接搜索如何打印完整的 `build` 日志吗，然后通过日志查看到底在哪一步失败了，于是刚好想起了前几天同学也发现过这样的问题，直接去问他。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gthd4su1tcj613l0u0jth02.jpg" alt="image-20210815123116551" style="zoom:50%;" />

### 打印build详细日志

波澜不惊，这样就可以了，嘿嘿嘿，好简单😅。

于是，改完，开始`build `....等待 loading

![image-20210815131524974](https://tva1.sinaimg.cn/large/008i3skNly1gtheeq1vfmj61l00980ug02.jpg)

内存不足，那就再改 `gradle` 内存，这下应该可以了吧，重启as,开始build...等待loading😑：

![image-20210815130912878](https://tva1.sinaimg.cn/large/008i3skNly1gthe8aqkrmj628y0m4jzj02.jpg)

这...,还是看不到具体日志啊，难道是真的看不了吗,花了10几分钟就这😤？



### gradlew assembleDebug

继续尝试别的方案,又是一顿搜索,这时候看到了 **StackOverFlow** 上有人用这个命令也可以，于是死马当活马医，继续尝试。

执行 **./gradlew clean assembleDebug** 开始尝试。结果如下：

![image-20210815151300208](https://tva1.sinaimg.cn/large/008i3skNly1gthht26rwyj622o0ao77e02.jpg)

我裂开了，于是继续找其他方案,来来回回折腾了快1个小时，还是这样，难不成我只能去对代码了吗？

太痛苦了，这时候只能寻求坐在我对面的开发组大佬帮助，希望能解决问题，阿门🙏。



让大佬来看了一下，大佬的回复很简单：

> 这应该已经是gradle能给出的最大提示了，你想要的错误具体位置，应该是无法打印出的，这种情况，你只能通过合并的diff对比下，看看是哪里导致的。

是啊，我忙活了这么久，居然忘记了最简单的，我直接去看git合并的diff啊，因为这个是 `inline` 的报错，那么范围也就只有 **改动过的inline** 啊，瞬间感觉自己走错方向了。

### 通过git-合并diff对比

直接 `github` 找到 `pr` ,点击 `Files changed` , `command+F` 搜索 `inline` ,瞬间流下来开心的泪水🤧.

![image-20210815152000980](https://tva1.sinaimg.cn/large/008i3skNly1gthi0ctny3j60nw0acmxb02.jpg)



果然范围小多了，那么接下来只需要找 `inline` 具体更改位置。

![image-20210815152305983](https://tva1.sinaimg.cn/large/008i3skNly1gthi3kfzj1j61960j6dip02.jpg)

于是乎，就发现上述代码，似乎不太对劲，乍一眼看上去没啥，但整个 `inline` 相关的更改里，只有这段新增了一个 `inline` 修饰。

于是试着去掉 `inline` ,发现居然好了，卧槽......

![APEX新赛季快被玩家玩坏了，靠表情包就能躲攻击，屏幕还能当外挂？](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRoaWgoOY997ra2DongBtQ6Xzqzql-qgEnlpw&usqp=CAU)

## 反思

到了这里，我似乎不敢相信自己，在之前的开发中，我从不知道 `inline` 修饰的方法，参数是函数对象时不可为 **null** ，于是我赶紧去搜了下相关文章：

**google**

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gthi7byq2gj61hc0p2aeb02.jpg" alt="image-20210815152642317" style="zoom:50%;" />

**度娘**

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gthi7o11u6j61k80l2n1v02.jpg" alt="image-20210815152702463" style="zoom:50%;" />



结果并没有发现相关有价值的文章，可能很少人像我那样操作，的确一般情况下，IDE 都会准确提示错误信息。

对比转换后的java代码，结果也是报错，也没有什么可奇怪的。

![image-20210815153106034](https://tva1.sinaimg.cn/large/008i3skNly1gthibwehppj625k0u0gy902.jpg)



于是接连测试了下：

结果也很简单。对于 `inline` 修饰的方法而言，如果方法参数是基本对象，那么可以为 **null** ,如果是 `函数对象(lambda)` ，则不可为 **null ** ,否则将引发编译错误。

![image-20210815154254706](https://tva1.sinaimg.cn/large/008i3skNly1gthio6lxddj60z20ee3zh02.jpg)

### 那到底为什么呢？

难道网上没有资料，这个问题就要烂在这里了吗，我不太甘心，既然没有现成，那我们就从 `inline` 的本质出发，寻找原因：

我们都知道，`inline` 的本质是在编译器将相关代码直接拷贝到了调用的地方，也就是说，比如我们上述截图中的 **testObj** 方法中的 `obj` 函数对象，其具体实现，会最终变成如下例子。

假设如下：

```kotlin
public final void testObj2(@Nullable Function0 obj) {
   int $i$f$testObj2 = 0;
}
```

> 里面的Function是具体可量化的接口，其是Kotlin提前定义好的。

但是现在，`obj函数对象` 可能为 **null**，即编译器没法确定了，编译器不知道这里到底应该复制什么玩意，如果不复制，那还怎么优化，但怎么复制，你都是 **null** 的，我怎么知道呢，所以直接 **null** 指针了。

当然上述过程我是猜测的，因为我没办法知道其内部原理，但是我相信实际和我推测的应该差别也不是很大。



## 碎碎谈

这虽然只是一个开发中不怎么常见的问题，但从整个过程而言，我的做法或许并不是很好，如果我一开始选择最直接的方式，效率会更高点，但也许我也会失去对一些东西的探索，之所以写本篇，很大程度上也是因为我在这个问题上所花费的时间让我必须写下来记录一下，希望这个过程可以帮助到大家。

大家加油，周末愉快！



