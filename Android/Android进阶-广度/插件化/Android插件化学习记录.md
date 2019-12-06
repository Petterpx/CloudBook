# Android插件化学习记录

#### 插件化出现背景

- app体积越来越大，大型APP模块多，向平台化靠拢。
- 模块耦合度高，协同开发沟通成本高
- 方法数可能会超过 65535，占用内存大

> 比如某个APP可能需要接入某个业务功能,而这个业务功能本身就是一个APP，这时候如果接入，意味着原业务团队需要维护两套APP，一个是本身自己，另一个是集成到这个APP中的，如果后续发现了问题，就需要修改两处APP，后续别的APP需要时，维护成本将极其高。



#### 解决方案

- 将一个大的 APK 按照业务与分隔成小的 apk
- 每个小的 apk 即可以独立运行又可以作为插件运行



#### 插件化优势

- 业务模块的解耦
- 提高团队开发效率
- 按需加载，内存占用更低等。



#### 插件化中的概念基础

- 宿主：主APP，可以加载插件，也称为Host
- 插件：插件App，被宿主加载的App,可以是跟普通App 一样的 Apk文件。
- 插件化：将一个应用按照宿主插件的方式改造就叫插件化。



#### 与组件化的对比

- 组件化是一种编程思想，而插件化是一种技术
- 组件化是为了代码的高度复用性而出现的
- 插件化是为了解决应用越来越庞大而出现的



#### 与动态更新的对比

- 与动态更新一样，都是动态加载技术的应用
- 动态更新是为了解决线上 bug 或 小功能的更新而出现
- 插件化是为了解决应用越来越庞大而出现的





### 插件化的原理，至少需要了解以下

- android ClassLoader加载Class文件原理
- Java 反射原理
- andrid 资源加载原理
- 四大组件加载原理



### 插件化解决的问题

#### Manifest处理方案

Gradle 在打包的时候，如果我们存在多个Model,那么它会将这些 Model 中的 Manifest合并，生成新的Merge Manifest.而插件化中需要解决的一个问题就是 	Manifest 中清单文件的合并。



#### 插件类如何加载？

首先区分是否是宿主APK,因为Android 会为已经安装的apk创建ClassLoader。

插件化会为每个插件apk创建classLoader。

![image-20191204150857578](https://tva1.sinaimg.cn/large/006tNbRwly1g9kphsl7f4j30kl07etbb.jpg)

简单理解就是·：

> 系统会为我们的宿主apk创建ClassLoader,而插件apk是由宿主app帮我们来创建的，创建完成之后才可以调用



#### 插件类加载

- 如何自定义ClassLoader加载类文件
- 如何调用插件Apk文件中的类



#### 插件类加载Demo

**宿主APP**

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        String apkPath = getExternalCacheDir().getAbsolutePath() + "/111.apk";
        loadApk(apkPath);
    }

    private void loadApk(String apkPath) {
        //只有我们自己的APP才可以访问这个App
        File optDir = getDir("opt", MODE_PRIVATE);
        //初始化classLoader
        DexClassLoader classLoader = new DexClassLoader(apkPath, optDir.getAbsolutePath(), null, this.getClassLoader());

        try {
            Class<?> aClass1 = classLoader.loadClass("com.cloud.bundle.MainActivity");
            if (aClass1 != null) {
                Object instacene = aClass1.newInstance();
                Method method = aClass1.getMethod("test");
                method.invoke(instacene);
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

我们加载的是另一个App中的某个类文件。



**插件APP**

```JAVA
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void test() {
        Log.e("Demo", "123");
        Toast.makeText(this, "sha", Toast.LENGTH_SHORT).show();
    }
}

```

不过我们简单的demo只能打印一个log,而涉及Toast相关都是无法完成的。



**使用adb push命令将插件app的apk推到我们应用相应的文件夹下。**

如下：

```java
 adb push /Users/petterp/Desktop/1111.apk /storage/emulated/0/Android/data/com.cloud.chajian/cache/111.apk
```



  adb push /Users/petterp/Desktop/123.apk /storage/emulated/0/Android/data/com.xiamuyao.androidassembly/files/123.apk



#### 自定义ClassLoader

```java
public class CustomClassLoader extends DexClassLoader {
    public CustomClassLoader(String dexPath, String optimizedDirectory, String librarySearchPath, ClassLoader parent) {
        super(dexPath, optimizedDirectory, librarySearchPath, parent);
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {

        byte[] classData = getClassData(name);

        if (classData != null) {
            return defineClass(name, classData, 0, classData.length);
        } else {
            throw new ClassNotFoundException();
        }
    }

    public byte[] getClassData(String name) {
        try {

            InputStream inputStream = new FileInputStream(name);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int buffersize = 4096;
            byte[] buffer = new byte[buffersize];
            int bytesNumRead = -1;
            while ((bytesNumRead = inputStream.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();

        } catch (Exception e) {

        }
        return null;
    }
}

```



#### 资源加载



#### 目前主流框架上的解决方案：



#### 插件化核心解决：

- 处理所有插件 apk 文件中的 Manifest文件。
- 管理宿主apk所有的插件 apk 信息
- 为每个插件 apk创建对应的类加载器，资源管理器

