# Small 插件化框架集成总结



#### 配置

注意，你的模块app名如无特殊需求不可更改

在你的 build.gradle下配置

```java
classpath 'net.wequick.tools.build:gradle-small:1.1.0-alpha2'
。。。


//在文件末尾引用
apply plugin: 'net.wequick.small'
small {
    aarVersion = '1.1.0-alpha2'
}
```



然后运行gradle脚本

```java
./gradlew small
```

![image-20191204225452005](https://tva1.sinaimg.cn/large/006tNbRwly1g9l2yl9whkj30jk08dq3l.jpg)



运行成功就这样。



#### 初始化

```java
public class CustomApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        //初始化Small
        Small.preSetUp(this);


    }
}
```



