# virtualapk集成总结

***声明，不支持Android x,不支持Android Tools Gradle 3.0.0以上***

[Github](https://github.com/didi/VirtualAPK)

集成教程



新建工程。

导入环境。



#### 宿主包配置：

App-build.gradle

```
apply plugin: 'com.didi.virtualapk.host'

dependencies {
    implementation 'com.didi.virtualapk:core:0.9.8'
}
```



项目build.gradle导入：

```
 dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0'
        classpath 'com.didi.virtualapk:gradle:0.9.8.4'
    }
```



设置你的 Application

```java
public class App extends Application{
    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        PluginManager.getInstance(base).init();
    }
}
```





#### 插件包设置

新建 Model-Phone & Tablet Model

具体配置如下：

Build.gradle

```java
dependencies{  
implementation 'com.didi.virtualapk:core:0.9.6'
}


apply plugin: 'com.didi.virtualapk.plugin'
virtualApk {
    packageId = 0x6f
    targetHost = '/Users/petterp/Desktop/testdidi/AndroidAssembly/app'
    applyHostMapping = true
}
```

其中 targetHost代表 宿主app 的路径;



宿主测试Activity

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loadPlugin(this);
        findViewById(R.id.btn_intent).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View mView) {
                Intent intent = new Intent();
              	//插件包名，Activity路径
                intent.setClassName("com.didi.test", "com.didi.test.TestActivity");
                startActivity(intent);

            }
        });
    }

    private void loadPlugin(Context base) {
        PluginManager pluginManager = PluginManager.getInstance(base);
      	//test.apk -> 生成的插件apk
        File apk = new File(getExternalCacheDir().getAbsolutePath() +"/test.apk");
        if (apk.exists()) {
            try {
                pluginManager.loadPlugin(apk);
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            Toast.makeText(getApplicationContext(),
                    "SDcard根目录未检测到plugin.apk插件", Toast.LENGTH_SHORT).show();
        }
    }

}
```



#### 如何生成插件apk?

先生成 宿主 包

![image-20191205234950456](https://tva1.sinaimg.cn/large/006tNbRwly1g9ma62ayzxj30is0niq98.jpg)



接着生成 插件 包

![image-20191205234933559](https://tva1.sinaimg.cn/large/006tNbRwly1g9ma5rk0n5j30ju0mc7at.jpg)



然后通过 adb 命令或者别的方式将插件包 传到 手机中。

最后就可以通过宿主app实现插件加载。



