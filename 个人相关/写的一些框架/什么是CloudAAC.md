

![image-20200908210016039](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/510fe1c57c844a4eab95c8f8e9ba39b9~tplv-k3u1fbpfcp-zoom-1.image)

> ***什么样的框架适合你？什么样的框架也许都不适合你。***



<br/>

与大千你我一样，皆是从 **无架构到MVC->模块化->MVP-> MVVM->AAC->组件化AAC。**

**很多时候，我有在考虑，我们真的需要过度去设计吗？可能有人喜欢 BaseVMFragnment<xxViewModel>,但有些时候，我们真的需要ViewModel吗，我们真的只有一个ViewModel吗，我可能真的不想去写<BaseViewModel>，对于2020的今天，带着这些问题，我开始思考，什么样的架构才是我们最合适的，适合于各类人士？，我想不出来，于是将选择主动权交给大家，并将过程中的一些想法通过代码汇聚于此，便于为大家提供思路，这就是CloudAAC，化繁为简，一个简易的组合式框架。**

***github:https://github.com/Petterpx/CloudAAC***



## 如何使用？

**导入依赖**

```groovy
allprojects {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```

```groovy
implementation 'com.github.Petterpx.CloudAAC:core:v1.0.3' 
```

**CloudAAC已经导入了以下组件：**

```
//一个非常优秀的状态栏处理工具
implementation 'com.gyf.immersionbar:immersionbar:3.0.0'
implementation 'com.gyf.immersionbar:immersionbar-ktx:3.0.0'

//Android-ktx扩展相关
implementation 'androidx.activity:activity:1.1.0'
implementation 'androidx.fragment:fragment-ktx:1.2.5'

//viewModel数据恢复
implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:2.2.0"
```



### **扩展支持**

> core模块仅提供了核心的基础类，选择将主动改造权交给了大家，为了更好的便于使用，CloudAAC 支持扩展 以下模块。

```groovy
implementation 'com.github.Petterpx.CloudAAC:databing_ktx:v1.0.3' 
implementation 'com.github.Petterpx.CloudAAC:viewbing_ktx:v1.0.3' 
implementation 'com.github.Petterpx.CloudAAC:tab_ktx:v1.0.3' 
```

<br/>

## 核心类解释

### Core
> 基础 **Base** 类。
- **BaseActivity**   *----基础BaseActivity类*
- **BaseFragment**  *----基础BaseFragment类*
- ...其他相关工具

<br/>

### Databing_ktx

> 适用于 **Databinding**  的通用 Activity && Fragment.
>
> **注意：** **binding** 变量 请谨慎使用，非必要场景下，**务必禁止使用，避免造成视图不一致的问题。**

- **BaseDataBingActivity<Bing>**      *----Activity-DataBing扩展*
- **BaseDataBingFragment<Bing>**     *----Fragment-DataBing扩展*
- **DataBingdinConfig**                  *---- DataBing的配置相关 (参考自 KunMinx）*

<br/>

### ViewBing_ktx

> 适用于 **ViewBing**  的通用 Activity && Fragment.

- **BaseViewBingActivity<Bing>**     *----Activity-ViewBing扩展*

- **BaseViewBingFragment<Bing>**     *----Fragment-ViewBing扩展*

- **BaseViewBingVMActivity<VM,Bing>**   

  > *Activity-ViewBing扩展,包含了默认的viewModel委托使用*

- **BaseViewBingVMFragment<VM,Bing>**    

  > *Fragment-ViewBing扩展,包含了默认的viewModel委托使用*

<br/>

### Tab_ktx (仍在优化中)

> 适用于主页 **tab** 的 扩展。

- base
  - **BasePagerAdapter**      
  - **BaseTabActivity**  
- ...其他相关工具

### ..._ktx_

*更多扩展等待加入，CloudAAC 尽可能采用扩展与组合方式，以便于不同人群的不同需求，当然如果你有更好的想法，欢迎 PR.*

<br/>

<br/>

**你知道的越多，你不知道的越多。并不提倡大家去频繁造轮子，但希望大家都能拥有去改造轮子的想法,CloudAAC 代码结构比较清晰，相关注释与边界已经注明，希望会对你会有所帮助。**




