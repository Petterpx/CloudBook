# 阿里Atlas学习指南



#### 框架原理：

![u1runtime_struct](https://tva1.sinaimg.cn/large/006tNbRwly1g9lw93n6mcj311s0nwwlm.jpg)

简要介绍如下

1. Hack工具层:

   包括了容器所需要的所有系统层面的注入和 hack 的工具初始化和效验，容器启动时先效验设备是否支持容器运行，不支持则采取降级并记录原因。

2. Bundle Framework 负责bundle 的安装 更新操作以及管理整个 bundle 的生命周期；

3. Runtime层：主要包括清单管理，版本管理，以及系统代理三大块，基于不同的触发点按需执行 bunde 的安装和加载；

   runtime层同时提供了开发期 debug和监控的两个模块。

   其中最重要的两个代理点如下：

   DelegateClassLoader:负责路由Class加载到各个 debug 内部

   DelegateResouce: 负责资源查找时能够找到 bundle 内的资源；

   其余的代理点都是主要给 上面这两个代理点服务。

4. 对外接入层：在框架构建的时候会自动替换 application,所以 Atlas 接入不需要初始化代码，所以apk下真正的application 是 AtlasBridgeApplication.除了完成框架的初始化，同时还内置了 **multidex** 的功能。

   原因如下

   - 很多大型的app不合理的初始化导致用multidex分包逻辑拆分的时候主dex的代码就有可能方法数超过65536，AtlasBridgeApplication与业务代码完全解耦，所以拆分上面只要保证atlas框架在主dex，其他代码无论怎么拆分都不会有问题；
   - 如果不替换Application，那么atlas的初始化就会在application里面，由于基于Atlas的动态部署实际上是类替换的机制，那么这种机制就会必然存在包括Application及其import的class等部分代码在dalvik不支持部署的情况，这个在使用过程中造成一定成本，需要小心的使用以避免dalivk内部class resolve机制导致部分class没成功，替换以后该问题得到最好的解决，除atlas本身以外，所有业务代码均可以动态部署；

   另外内置的原生的multidex在dalvik上面性能并不好，atlas内部对其进行了优化提高了在dalvik上面的体验。

   除AtlasBridgeApplication之外，接入层对外提供了部分工具类，包括主动install bundle，start bundle，以及获取全局的application等各种功能。





#### Bundle生命周期

![bundle_cycle](https://tva1.sinaimg.cn/large/006tNbRwly1g9lwtzw9bqj30qw0diadp.jpg)

**Installed** bundle被安装到storage目录

**Resolved** classloader被创建，assetpatch注入DelegateResoucces

**Active** bundle的安全校验通过；bundle的dex检测已经成功dexopt(or dex2oat)，resource已经成功注入

**Started** bundle开始运行，bundle application的onCreate方法被调用



#### 类加载机制

Atlas 会创建两种 ClassLoader。

第一种是 DelegateClassLoader，**作为类查找的一个路由器而存在**，本身并**不负责**真正类的加载，用于**替换**原有的 PathClassLoader.

第二种是 BundleClassLoader,在每个插件被解析时会分配一个 BundleClassLoader为 parent,同时在路由过程中能够找到所有 bundle 的classLoader;

BundleClassLoader 以BootClassLoader为parent(父加载器),同时引用PathClassLoader,BundleClassLoader自身 加载的过程如下：

1. findOwn: 先查找 bundle dex自身内部的 class
2. findDependency：查找bundle 依赖得的bundle 内的class
3. findPath：查找主apk中的class

![classloader](https://tva1.sinaimg.cn/large/006tNbRwly1g9lx3pfcihj30re0eqnns.jpg)



