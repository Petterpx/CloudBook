# 关于Gradle,你应该了解的这些事

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1gzuo5lj9hrj20xd0a6gmz.jpg)

Hi,很高兴见到你！

本篇是Gradle系列第三篇，从这篇开始，我们将开始真正的Gradle研究之旅，抓紧喽！

~~本篇原本的主题是，关于Gradle,你应该了解的这些事~~

~~但仔细想了想，我们真的需要关心那么多吗？术业有专攻，不懂就google,我并不想制造焦虑，所以对于Gradle,大家可能应该明白哪些，而哪些不需要明白，根据兴趣而定。~~

本篇不同于前两篇的实践，严格意义上算一篇[全面且详细]的科普文。作为和大家一样的gradle初学者，我尽量说的更直白一点，希望看完之后，你们也能自然而然的明白以下几点：

- Gradle是什么？
- Gradle和AGP(Android Gradle)之间的关系？
- Gradle依赖管理傻傻分不清
- Gradle的生命周期
- Task和插件有什么关系
- 相关的APT和ASM和Gradle到底有啥关系

## 什么是Gradle？

Gradle是一个开源的自动化构建工具，其构建脚本使用Groovy或者Kotlin编写。作为一个Android开发者，我们应该天天在使用，就长下面这样，当我们点击build.gradle时，如下图所示：

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1gzvabyda65j21220kq40a.jpg" alt="image-20220302093416878" style="zoom:33%;" />

## Gradle和AGP之间的关系？

我们常使用的Android Studio构建系统其主要以Gradle为基础。在Android为了满足构建App,于是Android团队写了一个相应的Android Gradle插件，这个插件名字就又被我们简称为 `AGP` 。

> 所以AGP和Gradle两者的关系就是 `插件` 与 `基础构建工具` 的区别。
>
> 具体一点就是：
>
> Gradle作为一个基础构建工具，在Android中为了构建Apk,Android团队开发了AGP插件以满足使用。这和我们平时用的其他Gradle插件是一样的,不同之处就在于功能不同。

如下图所示：

![image-20220302094129616](https://tva1.sinaimg.cn/large/e6c9d24ely1gzvajgiprmj216c038aas.jpg)

### 版本问题🧐？

如果你使用的不是较新的agp版本7.x，而是是4.x, 你可能比较疑惑，为什么这个版本号和我们使用的Gradle版本号(gradle-wrapper.properties)不一致？

官方的回答是这样的：

> AGP 的版本号原本主要和Android Studio关联，一般而言，每次Android Studio更新，相应的AGP插件也会更新，所以在你升级Android Studio 时，一般也会收到插件的更新提示。当你每次更新AGP插件版本时，Android Studio也会提示推荐你更新Gradle版本，以兼容更新。

但自2020年11月开始，AGP的版本号命名规则进行了变更，原文如下：

- AGP 现在将使用语义版本控制，并且重大变更将在主要版本中发布。
- 每年将发布一个 AGP 主要版本，与 Gradle 主要版本保持一致。
- AGP 4.2 之后版本为版本 7.0，并且会要求升级到 Gradle 7.x 版。AGP 的每个主要版本都会要求在底层 Gradle 工具中进行主要版本升级。
- API 的弃用将提前大约一年进行，同时提供替换功能。废弃的 API 将在大约一年后的下次重大更新期间移除。

### 谨慎升级😟

如果你现在使用的4.2的agp,如果使用的较新的AS，在Android Studio里就会提示建议你升级到7.x,但是这对实际项目而言并不是一个简单的事，主要在于我们项目中如果使用了其他的插件，可能就会产生其他的工作量，所以在升级之前，建议看看必需的插件是否已经支持7.x，这点尤为重要，所以一般大厂的agp版本都不是很高，4.x已经算很新了。

## Gradle的依赖管理



## Gradle的生命周期

Gradle的构建具有三个不同阶段：

Initialization(初始化) -> Configuration(配置) -> Execution(执行) 

### Initialization

看看这里 

https://blog.csdn.net/sbsujjbcy/article/details/52079413

这个阶段主要目的是初始化构建，分为了两个子过程，执行init Script 与 Setting Script。

Gradle本身支持单项目与多项目构建。在初始化时，Gradle会去确定哪些项目参与构建，并为每一个项目创建一个 Project 实例。

这个阶段会创建settings对象，并执行settings.gradle脚本，建立工程之间的层次关系。

> 需要注意的的是，buildSrc总是优先被执行的。



### Configuration

这个阶段，Gradle分别会在每个Project对象上执行对应的build.gradle脚本，对project进行配置.并且会根据项目的依赖关系，基于有向无环图的方式生成一个依赖关系图，如下所示：

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h04jrzdrnkj21ep0u040y.jpg" alt="img" style="zoom:33%;" />

> 什么是有向无环图？
>
> Directed acyclic graph (DAG)，即这个图的边必须有方向，图内无环。
>
> 有向指的就是你的微信好友把你删除了，你此时不会主动得知，这就是单向；
>
> 环的意思就是，从一张图上任意点出发，都能回到起始点。所以
>
> 这样的一个图而言，是没法找到正确的打开方式；
>
> 综合起来就是 对于一张图，从任一顶点出发，都无法回到该点，则这就是一个有向无环图，简称图中没有回路(环的)有向图。
>
> 具体参见这篇：[Android 启动优化（一） - 有向无环图](https://juejin.cn/post/6926794003794903048)

### Execution

在执行阶段，Gradle会判断配置阶段创建的那些Task都需要被执行，然后执行选中的每个Task。

### 监听生命周期

在gradle的构建阶段，我们的构建脚本可以接收相应的通知，通常有两种通知的方式



## Task和插件有什么关系

我们所使用的的Gradle只是一个框架，其定义了基本的构建流程与规则，并且向外暴露出了一些时机，对于不同的开发场景或者业务场景，我们所需要的功能 不同，比如在Android中，默认已经为我们依赖了Android Plugin的插件，以便完成工程的打包，而每一个插件中都可能存在着多个任务，这个任务又叫做task,即Gradle的最小执行单元。

你可以创建一个插件，插件内部可能包含了多个Task,或者单独创建一个task在其他model中进行依赖，插件就类似一个包含作用。
