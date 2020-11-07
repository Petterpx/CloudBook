



#  JetPack-**Lifecycle**

Lifecycle是在Android support26开始就被引入的，所以我们在Activity或者Fragment中经常都可以看到，甚至不需要过多依赖都可以用到。

### Lifecycle用来干什么？

解决Android常见的内存泄漏与更简单的生命周期管理。

比如：

在我们的网络请求中，我们一般不会直接在Activity或者Fragment onDestory 中直接去写销毁网络请求的代码，一般都是通过一个代理或者回调以便在数据类中销毁我们的网络请求。但是如果



git remote add github https://github.com/people-rmxc/Bullet-Ktx.git