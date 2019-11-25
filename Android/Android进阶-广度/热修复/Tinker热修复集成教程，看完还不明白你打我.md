# Tinker热修复集成教程，看完还不明白你打我

[Tinker官网：](http://www.tinkerpatch.com/)

写这篇博客的目的是，很多刚接触热修复的同学在了解了什么是热修复，及要选用tinker作为开发或者demo实践中。解决新同学集成过程中的迷茫。

热修复是什么，我就不过多解释了，为什么要集成Tinker我想大家也都知道，我这里只是教大家如何快速集成并使用，至于与其他热修复框架的对比，暂时也没有使用过其他热修复框架，所以也就不发一张所谓的对比图来告诉大家如何如何，并没有什么实际意义。

目前网上关于集成 Tinker 的教程大多数都是17年时候的操作，现在都9102年了，有些19年的博客结果还是用的以前的操作手法，旧操作没什么不对，但至少是在写博客，也至少告诉大家啊。都是各种抄袭，别人这么做，我也这么做，真的很难找到一个是自己看着官网做的(官方github的集成也是17年的。。。。em)。

碍于这些原因，催生了这篇博文，好了，我们开始吧。



***测试背景：***

> Android Studio **3.5.2**
>
> SDK **29**
>
> Gradle **5.4.1**
>
> 测试设备 **小米5s Plus，Android8.0**



## 两种方式集成：

目前已知的集成方式主要有两种，一种是傻瓜式，就是复制粘贴，另一种是稍微多花几步，区别就在于Application是反射还是自己添加。



### 傻瓜式集成：

![image-20191122141044785](https://tva1.sinaimg.cn/large/006y8mN6ly1g96sdinjo7j314i04aq3u.jpg)

进入官网，按照文档开始进行操作。

![image-20191122133906261](https://tva1.sinaimg.cn/large/006y8mN6ly1g96rglz9elj31gq0u0n62.jpg)



> 为了防止官网有些话语难理解，贴心的为大家一步步来。

为了便于统一版本，后期好管理，我们先打开 gradle.propertites.配上如下两句话

```java
TINKER_APPLICATION=1.9.14
TINKER_VERSION=1.2.14
```

![image-20191122134225363](https://tva1.sinaimg.cn/large/006y8mN6ly1g96rk3e720j321t0s8wsj.jpg)

目前tinker最新的版本就是如上，如果后期更新，再改呗。



然后打开 build.gradle

增添如下： ps:*刚才配置的统一版本这时候用处就出来了*

```
classpath "com.tinkerpatch.sdk:tinkerpatch-gradle-plugin:${TINKER_VERSION}"
```

![image-20191122134636716](https://tva1.sinaimg.cn/large/006y8mN6ly1g96rof1qvoj320c0cm42k.jpg)



然后打开你app目录下的build.gradle

增添如下三句：

> ```java
> apply from: 'tinkerpatch.gradle'
>   
>   ...
>   
> //tinker核心库
> implementation("com.tinkerpatch.sdk:tinkerpatch-android-sdk:${TINKER_VERSION}"){ changing = true }
> //分包，为了多model准备
> implementation 'com.android.support:multidex:1.0.3'
> ```

![image-20191122135052353](https://tva1.sinaimg.cn/large/006y8mN6ly1g96rsw8br1j31qz0u0k4e.jpg)

上面的 apply from...，其实就是相当于导入了某个配置文件。为啥要这样做呢？不这样写难不成我将所有东西都写到 build.gradle，不复杂吗。。。。em



tinkerpatch.gradle是tinker已经提供好的，下载复制粘贴即可。。。

```java
apply plugin: 'tinkerpatch-support'

/**
 * TODO: 请按自己的需求修改为适应自己工程的参数
 */
//apk的生成目录
def bakPath = file("${buildDir}/bakApk/")
def baseInfo = "基准包的文件夹名"
def variantName = "debug"

/**
 * 对于插件各参数的详细解析请参考
 * http://tinkerpatch.com/Docs/SDK
 */
tinkerpatchSupport {
    /** 可以在debug的时候关闭 tinkerPatch **/
    /** 当disable tinker的时候需要添加multiDexKeepProguard和proguardFiles,
     这些配置文件本身由tinkerPatch的插件自动添加，当你disable后需要手动添加
     你可以copy本示例中的proguardRules.pro和tinkerMultidexKeep.pro,
     需要你手动修改'tinker.sample.android.app'本示例的包名为你自己的包名, com.xxx前缀的包名不用修改
     **/
    tinkerEnable = true

    /** 反射接入Application*/
    reflectApplication = true

    /**
     * 是否开启加固模式，只能在APK将要进行加固时使用，否则会patch失败。
     * 如果只在某个渠道使用了加固，可使用多flavors配置
     **/
    protectedApp = false

    /**
     * 实验功能
     * 补丁是否支持新增 Activity (新增Activity的exported属性必须为false)
     **/
    supportComponent = false

    /** 每次编译产生的apk/mapping.txt/R.txt 归档存储的位置*/
    autoBackupApkPath = "${bakPath}"

    appKey = "你自己的key"

    /** 注意: 若发布新的全量包, appVersion一定要更新 **/
    appVersion = "版本号，对应tinker管理app那里填的版本"

    def pathPrefix = "${bakPath}/${baseInfo}/${variantName}/"
    def name = "${project.name}-${variantName}"

    /** 基准包的文件路径, 对应 tinker 插件中的 oldApk 参数;编译补丁包时，必需指定基准版本的 apk，默认值为空，则表示不是进行补丁包的编译。*/
    baseApkFile = "${pathPrefix}/${name}.apk"
    /**
     * 基准包的 Proguard mapping.txt 文件路径, 对应 tinker 插件 applyMapping 参数；在编译新的 apk 时候，
     * 我们希望通过保持基准 apk 的 proguard 混淆方式，从而减少补丁包的大小。
     * 这是强烈推荐的，编译补丁包时，我们推荐输入基准 apk 生成的 mapping.txt 文件。
     * */
    baseProguardMappingFile = "${pathPrefix}/${name}-mapping.txt"

    /**
     * 基准包的资源 R.txt 文件路径, 对应 tinker 插件 applyResourceMapping 参数；
     * 在编译新的apk时候，我们希望通基准 apk 的 R.txt 文件来保持 Resource Id 的分配，这样不仅可以减少补丁包的大小，
     * 同时也避免由于 Resource Id 改变导致 remote view 异常。
     * */
    baseResourceRFile = "${pathPrefix}/${name}-R.txt"

    /**
     *  若有编译多flavors需求, 可以参照： https://github.com/TinkerPatch/tinkerpatch-flavors-sample
     *  注意: 除非你不同的flavor代码是不一样的,不然建议采用zip comment或者文件方式生成渠道信息（相关工具：walle 或者 packer-ng）
     **/
}

/**
 * 用于用户在代码中判断tinkerPatch是否被使能
 */
android {
    defaultConfig {
        buildConfigField "boolean", "TINKER_ENABLE", "${tinkerpatchSupport.tinkerEnable}"
    }
}

/**
 * 一般来说,我们无需对下面的参数做任何的修改
 * 对于各参数的详细介绍请参考:
 * https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97
 */
tinkerPatch {
    ignoreWarning = true
    useSign = true
    dex {
        dexMode = "jar"
        pattern = ["classes*.dex"]
        loader = []
    }
    lib {
        pattern = ["lib/*/*.so"]
    }

    res {
        pattern = ["res/*", "r/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
        ignoreChange = []
        largeModSize = 100
    }

    packageConfig {
    }
    sevenZip {
        zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
//        path = "/usr/local/bin/7za"
    }
    buildConfig {
        keepDexApply = false
    }
}
```

如果你直接复制上面的代码的话，在你的app底下新建一个file,命名tinkerpatch.gradle,（命名随意，注意你build里面导入时也要改）。然后粘贴代码进去即可。

> 注意上面的一些参数：
>
> reflectApplication =true.
>
> 正因为官网提供了反射创建Application.所以我们免除了一些别的步骤，不过这种方式并不是官方推荐，因为存在一些加载不成功的几率，而且因为反射对性能的消耗，所以默认是false.



然后Tinker贴心的已经为我们准备好了 Application,老样子，复制粘贴即可

> 虽然很长，但是除了initTinkerPatch 方法，剩下的都是官方对接口的一些解释。[具体参考](http://www.tinkerpatch.com/Docs/api)

```java
public class SampleApplication extends Application {
    private static final String TAG = "Tinker.SampleApppatch";

    private ApplicationLike tinkerApplicationLike;

    public SampleApplication() {

    }

    @Override
    public void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        //you must install multiDex whatever tinker is installed!
        //MultiDex.install(base);
    }


    /**
     * 由于在onCreate替换真正的Application,
     * 我们建议在onCreate初始化TinkerPatch,而不是attachBaseContext
     */
    @Override
    public void onCreate() {
        super.onCreate();
        initTinkerPatch();
    }

    /**
     * 我们需要确保至少对主进程跟patch进程初始化 TinkerPatch
     */
    private void initTinkerPatch() {
        // 如果开启tinker
        if (BuildConfig.TINKER_ENABLE) {
            tinkerApplicationLike = TinkerPatchApplicationLike.getTinkerPatchApplicationLike();
            // 初始化TinkerPatch SDK
            TinkerPatch.init(tinkerApplicationLike)
                    .reflectPatchLibrary()
                    .setPatchRollbackOnScreenOff(true)
                    .setPatchRestartOnSrceenOff(true)
                    //后台访问时间间隔，(-1)代表以后都不再访问
                    .setFetchPatchIntervalByHours(1)
                    //每次都向后台请求补丁包更新
                    .fetchPatchUpdate(true)
                    //合成后回调
                    .setPatchResultCallback(new ResultCallBack() {
                        @Override
                        public void onPatchResult(PatchResult patchResult) {
                            Log.e("Demo", "合成成功");
                        }
                    })
                    //锁屏清除补丁
                    .setPatchRollbackOnScreenOff(true)
                    .setPatchResultCallback(new ResultCallBack() {
                        @Override
                        public void onPatchResult(PatchResult patchResult) {
                            Log.e("Demo", "已经清除了所有补丁");
                        }
                    });
            // 获取当前的补丁版本
            Log.d(TAG, "Current patch version is " + TinkerPatch.with().getPatchVersion());
            // fetchPatchUpdateAndPollWithInterval 与 fetchPatchUpdate(false)
            // 不同的是，会通过handler的方式去轮询
            TinkerPatch.with().fetchPatchUpdateAndPollWithInterval();
        }
    }

    /**
     * 在这里给出TinkerPatch的所有接口解释
     * 更详细的解释请参考:http://tinkerpatch.com/Docs/api
     */
    private void useSample() {
        TinkerPatch.init(tinkerApplicationLike)
                //是否自动反射Library路径,无须手动加载补丁中的So文件
                //注意,调用在反射接口之后才能生效,你也可以使用Tinker的方式加载Library
                .reflectPatchLibrary()
                //向后台获取是否有补丁包更新,默认的访问间隔为3个小时
                //若参数为true,即每次调用都会真正的访问后台配置
                .fetchPatchUpdate(false)
                //设置访问后台补丁包更新配置的时间间隔,默认为3个小时
                .setFetchPatchIntervalByHours(3)
                //向后台获得动态配置,默认的访问间隔为3个小时
                //若参数为true,即每次调用都会真正的访问后台配置
                .fetchDynamicConfig(new ConfigRequestCallback() {
                    @Override
                    public void onSuccess(HashMap<String, String> hashMap) {

                    }

                    @Override
                    public void onFail(Exception e) {

                    }
                }, false)
                //设置访问后台动态配置的时间间隔,默认为3个小时
                .setFetchDynamicConfigIntervalByHours(3)
                //设置当前渠道号,对于某些渠道我们可能会想屏蔽补丁功能
                //设置渠道后,我们就可以使用后台的条件控制渠道更新
                .setAppChannel("default")
                //屏蔽部分渠道的补丁功能
                .addIgnoreAppChannel("googleplay")
                //设置tinkerpatch平台的条件下发参数
                .setPatchCondition("test", "1")
                //设置补丁合成成功后,锁屏重启程序
                //默认是等应用自然重启
                .setPatchRestartOnSrceenOff(true)
                //我们可以通过ResultCallBack设置对合成后的回调
                //例如弹框什么
                //注意，setPatchResultCallback 的回调是运行在 intentService 的线程中
                .setPatchResultCallback(new ResultCallBack() {
                    @Override
                    public void onPatchResult(PatchResult patchResult) {
                        Log.i(TAG, "onPatchResult callback here");
                    }
                })
                //设置收到后台回退要求时,锁屏清除补丁
                //默认是等主进程重启时自动清除
                .setPatchRollbackOnScreenOff(true)
                //我们可以通过RollbackCallBack设置对回退时的回调
                .setPatchRollBackCallback(new RollbackCallBack() {
                    @Override
                    public void onPatchRollback() {
                        Log.i(TAG, "onPatchRollback callback here");
                    }
                });
    }

    /**
     * 自定义Tinker类的高级用法, 使用更灵活，但是需要对tinker有更进一步的了解
     * 更详细的解释请参考:http://tinkerpatch.com/Docs/api
     */
    private void complexSample() {
        //修改tinker的构造函数,自定义类
        TinkerPatch.Builder builder = new TinkerPatch.Builder(tinkerApplicationLike)
                .listener(new DefaultPatchListener(this))
                .loadReporter(new DefaultLoadReporter(this))
                .patchReporter(new DefaultPatchReporter(this))
                .resultServiceClass(TinkerServerResultService.class)
                .upgradePatch(new UpgradePatch())
                .patchRequestCallback(new TinkerPatchRequestCallback());
        //.requestLoader(new OkHttpLoader());

        TinkerPatch.init(builder.build());
    }
}
```



然后配置你的AndroidManifest，如下

![image-20191122140240940](https://tva1.sinaimg.cn/large/006y8mN6ly1g96s55rofbj31he0u0wlu.jpg)



写到这里，傻瓜式配置就算完了，简单吧，接下来用第二种方式接入，稍微麻烦一点。









最后，希望大家在学习的过程中，多去参考官方的教程，有时候官方的确很坑，但是当你解决了的时候，发现官方写的其实很明白，只是我们没理解。当你再给别的同学分享你的经验的时候，希望大家能说明白为什么我要这样做？而不是按照自己的思路去操作，考虑一下新同学的感受。



