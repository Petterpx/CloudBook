# Android NDK开发初试

首先，什么是NDK开发，听到这个词，我的第一感觉是高大上的，其实倒也没错，NDK在Android开发中属于偏底层的，需要与C++等进行联系，它没有像应用层开发那么灵活，但是作为开发者，我们必须了解其简单的使用，及为什么要使用它？

说起NDK，不得不提的一个关键点就是JNI。

JNI诞生的本意是为了方便 Java调用C，C++等本地代码而封装的一层接口。我们都知道。Java的优点是跨平台，但是作为优点的同时，在于本地交互的是后续就出现了短板。Java的跨平台特效导致其本地交互的能力不够强大，一些和操作系统相关的特效Java无法完成，于是Java提供了 JNI专门用于和本地代码教会，这样就增强了 Java 语言的本地交互能力。通过Java JNI，用户可以调用 C，C++等相关代码。

NDK是Android所提供的一个工具集合，通过NDK可以在Android中更加方便的通过JNI来访问本地代码，比如C或者C++。NDK还提供了 交叉编译器(在一种平台上编译，编译出来的程序，是放到别的平台上运行即编译的环境，和运行的环境不一样，属于交叉的，此所谓cross。),开发人员只需要简单修改 mk 文件就可以生成特定cpu 平台的动态库。使用ndk有以下好处：

- 提高代码的安全性。(so包很难反编译)
- 可以很方便的使用目已有的C/C++开源库 (比如音视频开发使用ffmpeg)
- 便于平台之间的移植。(常见于算法移植，或者某个智能控制系统 android端控制产品落地)
- 提高程序在某些特定条件下的执行效率，但是并不能明显提升 Android程序效率。(也很好理解，某个业内领先算法，很可能是C++或者C写的，这时候某些场景下使用其处理逻辑会更好，在并没有sdk的情况下，这时候就需要通过ndk调用此so包)



### 那么接下来就开始我们的主操作：

Android Studio 3.5

现在我们想创建一个ndk的项目，真的是简单很多，无需要太多的过程，直接创建即可。如果你使用mac，那么将更简单。

首先新创建项目，选择 Native C++，然后一路傻瓜式，没说的

![image-20190919200840943](https://tva1.sinaimg.cn/large/006y8mN6ly1g75327yo19j311e0u00x6.jpg)

一顿build之后，我们看看现在的项目目录

![image-20190919202022743](https://tva1.sinaimg.cn/large/006y8mN6ly1g753ewtppfj30ko0lgaba.jpg)

多了一个叫做 cpp的文件夹，然后我们具体关注 CMakeLists.txt和 native-lib这两个文件。

我们先看MainActivity示例代码：

```java
public class MainActivity extends AppCompatActivity {

  	//加载so包
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(stringFromJNI());
    }

  
  	//相应的 native方法
    public native String stringFromJNI();

}
```

看起来也没有什么，我们的关注点放在注释的地方，了解其意即可：



接着我们看看这个 stringFromJNI具体指的是哪个函数

回到native-lib文件，这是一个c++的代码：

```c++
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_androidndktest_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

里面也很简单，就是返回了一个字符串。关于C++的语法，需要ndk开发的同学就需要掌握其中了，因为偶尔如果出现bug，搞不好就得翻一遍具体实现了。如果只是简单了解ndk开发，那么关于C++语法简单明白即可，因为我们一般使用的时候，其中的代码我们并不需要知道具体如何实现，只是负责调用。

首先我们要知道jni中函数名的命名规范：

> Java_com_example_androidndktest_MainActivity_stringFromJNI
>
> **Java_包名_类名_方法名**

然后很多人肯定会对这句话感到懵逼：**extern "C" JNIEXPORT jstring JNICALL**

> **extern "C"** 它指定内部的函数采用C语言的命名风格来编译，否则当 JNI 采用 C++来实现时，由于 c 和 C++编译过程中对函数的命名风格不同，这将导致 JNI 在链接时无法根据函数名查找到具体的函数，那么 JNI 

> **extern "C"**
>
> 指定内部的函数采用C语言的命名风格来编译，否则当 JNI 采用C++来实现时，由于 C 和 C++ 编译过程中对函数的命名风格不同，这将导致 JNI 在链接时无法根据 函数名找到具体的函数，那么JNI 调用就无法完成。
>
> **JNIEXPORT**  **JNICALL** 它们是 JNI 中定义的宏，可以在 jni.h 这个头文件中找到
>
> **jstring** 一种数据类型转换，C



好了现在我们点击运行，查看屏幕结果：

![image-20190919205149089](https://tva1.sinaimg.cn/large/006y8mN6ly1g754b3sbmrj30m61amn3l.jpg)



嗯，很简单啊，当然在Android Studio强大加持下，在以前eclipse 时代的很多操作，我们现在都避免了，所以看起来很简单。现在我们自己新建一个 cpp文件试试,然后将默认native-lib中的代码复制过来，并更改函数名和其中的String字符串。

![image-20190919205529000](https://tva1.sinaimg.cn/large/006y8mN6ly1g754ex0ygaj32jo0oiq9j.jpg)



全部报红了对吧，我们现在查看 **CMakeList.tx**t 文件

```java
cmake_minimum_required(VERSION 3.4.1)

add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             native-lib.cpp

        )

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )
。。。

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

我省略了其中的注释，现在我们查看具体关键字：

**add_library(**

**JniTest-lib**

**SHARED**

 **JniTest.cpp)**

第一个参数是库的名称，这个名称由我们自己来取，第二个是模式，第三个是你要编译的C++代码，新添加的C++文件，需要在这里添加。

我们将我们刚才新建的文件，添加到这里，如下；

```java
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        native-lib.cpp

        native-test.cpp

        )
```



Sync new一下，我们再去MainActivity中,添加我们自己的方法，并调用,如下：

```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(stringTest());
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI();
    
    public native String stringTest();

}
```



运行一下，看效果

![image-20190919210612479](https://tva1.sinaimg.cn/large/006y8mN6ly1g754q2pet3j30m61amn3l.jpg)

一个最简单的 ndk 使用demo就出来了，关于更多的使用，如果以后涉及，我也会写出来，这篇只是基础上让大家有这样的一个思想，ndk与jni之间的关系，及在android中如何使用。如果对你有所帮助，不胜荣幸。