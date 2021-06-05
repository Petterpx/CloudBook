# 浅谈Android热更新的前因后果

> 最近一个月本该来说时间应该最多，但却没有写几篇博客，真是惭愧啊。最近在集成热修复，正好下周一要进行技术分享，所以就来好好梳理一下 热修复是个什么吧。

**首先，我们需要持有以下几个问题:**

1. 什么是热修复？它可以帮我解决什么问题？
2. 热修复的产生背景？
3. 热修复的基本原理是什么？
4. 如何选择热修复框架？
5. 热修复的注意事项
6. ~~热修复与多渠道？~~
7. ~~自动化构建与热修复？~~

> 上面一共有7个问题，如果是新同学的话，后面两条可能不会很了解，建议自行补课学习。于是最基本的5个问题，我们必须明白，这是我们每个开发者学习一个新知识的基本需要做到的。



### 什么是热修复？它可以帮我解决什么问题？

其实简单来说，热修复就是一种动态加载技术，比如你线上某个产品此时出现了bug：

> 传统流程：**debug->测试->发布新版 ->用户安装**(各平台审核时间不一，而且用户需要手动下载或者更新)
>
> 集成热修复情况下：**dubug->测试->推送补丁->自动下载补丁修复** (用户不知情况，自动下载补丁并修复)

对比下来，我们不难发现，传统流程存在这几大弊端：

1. 发版代价大
2. 用户下载安装的成本过高
3. bug修复不及时，取决于各平台的审核时间等等



### 热修复产生背景？

- app发版成本高
- 用H5集成某些经常变动的业务逻辑，但这种方案需要学习成本，而且对于无法转为H5形式的代码仍旧是无法修复；
- Instant Run

上面三个原因中，我们主要来谈一下 Instant Run:

> Android Studio2.0时，新增了一个 Instant Run的功能，而各大厂的热修复方案，在代码，资源等方面的实现都是很大程度上参考了Instant Run的代码。所以可以说 Instant Run 是推进Android 热修复的主因。
>
> 那 Instant Run内部是如何做到这一点呢？
>
> 1. 构建一个新的 AssetManager(资源管理框架),并通过反射调用这个 addAssetPath，把这个完整的新资源加入到 AssetManager中，这样就得到了一个含有所有新资源的 AssetManager.
> 2. 找到所有之前引用到原有AssetManager的地方，通过反射，把引用出替换为新的AssetManager.
>
> **参考自 <深入探索Android热修复技术原理>**

[关于 InstantRun 的更多解释请参考：](https://cloud.tencent.com/developer/article/1037769)



### 热修复的原理是什么？

我们都知道热修复都相当于动态加载，那么动态加载到底动态在哪里了呢。

说到这个就躲不过一个关键点 **ClassLoader(类加载器)** ,所以我们先从Java开始。

![image-20191124213728766](https://tva1.sinaimg.cn/large/006y8mN6ly1g99gizkazwj30wz0brjui.jpg)

我们都知道Java的类加载器有四种，分别为：

- Bootstarp ClassLoader 
- Extension ClassLoader
- App ClassLoader  加载应用ClassLoader
- Custom ClassLoader 加载自己的class文件

类加载过程如下：

> 过程： **加载-连接(验证-准备-解析)-初始化**
>
> 1. 加载
>
>    将类的信息(字节码)从文件中获取并载入到JVM的内存中
>
> 2. 连接
>
>    验证：检查读入的结构是否符合JVM规范
>
>    准备：分配一个结构来存储类的信息
>
>    解析：将类的常量池中的所有引用改变成直接引用
>
> 3. 初始化
>
>    执行静态初始化程序,把静态变量初始化成指定的值

其中用到的三个主要机制：

> 1. 双亲委托机制
> 2. 全盘负责机制
> 3. 缓存机制

其实后面的两个机制都是主要从双亲委托机制延续而来。详细的[Java类加载请参考我的另一篇博客](https://blog.csdn.net/petterp/article/details/88757059)

***在说明了Java 的ClassLoader之后，我们接下来开始Android的ClassLoader，不同于Java的是，Java中的ClassLoader可以加载 jar 文件和 Class文件，而Android中加载的是Dex文件，这就需要重新设计相关的ClassLoader类。所以Android 的ClassLoader 我们会说的详细一点***

![image-20191125191625071](https://tva1.sinaimg.cn/large/006y8mN6ly1g9ai4rgwapj31ew0qk42t.jpg)

### 源码解析

***在这里，顺便提一下，这里贴的代码版本是Android 9.0,在8.0以后，PathClassLoader和DexClassLoader并没有什么区别，因为唯一的一个区别参数 optimizedDirectory已经被废弃。***

首先是 loadClass，也就是我们类加载的核心方法方法：

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
        // First, check if the class has already been loaded
        //查找当前类是否被加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                		//查看父加载器是否加载过
                    c = parent.loadClass(name, false);
                } else {
                		//如果没有加载过，调用根加载器加载，双亲委托模式的实现
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
						
						//找到根加载器依然为null,只能自己加载了
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                c = findClass(name);
            }
        }
        return c;
}
```

> 这里有个问题，JVM双亲委托机制可以被打破吗？先保留疑问。



我们主要去看他的 **findClass**方法

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}
```

这个方法是一个null实现，也就是需要我们开发者自己去做。

从上面基础我们知道，在Android中，是有 PathClassLoader和 DexClassLoader，而它们又都继承与 BaseDexClassLoader,而这个BaseDexClassLoader又继承与 ClassLoader，并将findClass方法交给子类自己实现，所以我们从它的两个子类 PathClassLoader和 DexClassLoader入手，看看它们是怎么处理的。

这里碍于Android Studio无法查看相关具体实现源码，所以我们从源码网站上查询：

#### [PathClassLoader](http://androidxref.com/9.0.0_r3/xref/libcore/dalvik/src/main/java/dalvik/system/PathClassLoader.java)

```java
public class PathClassLoader extends BaseDexClassLoader {
    
    public PathClassLoader(String dexPath, ClassLoader parent) {
        super(dexPath, null, null, parent);
    }
  
   // dexPath: 需要加载的文件列表，文件可以是包含了 classes.dex 的 JAR/APK/ZIP，也可以直接使用 classes.dex 文件，多个文件用 “:” 分割
 // librarySearchPath: 存放需要加载的 native 库的目录
 // parent: 父 ClassLoader
    public PathClassLoader(String dexPath, String librarySearchPath, ClassLoader parent) {
        super(dexPath, null, librarySearchPath, parent);
    }
}

```

由注释看可以发现PathClassLoader被用来加载本地文件系统上的文件或目录，因为它调用的 BaseDexClassLoader的第二个参数为null，即未传入优化后的Dex文件。

注意：**Android 8.0之后，BaseClassLoader第二个参数为(optimizedDirectory)为null,所以DexClassLoader与PathClassLoader并无区别**



#### [DexClassLoader](http://androidxref.com/9.0.0_r3/xref/libcore/dalvik/src/main/java/dalvik/system/DexClassLoader.java)

![image-20191121000739855](https://tva1.sinaimg.cn/large/006y8mN6ly1g9a3y3a4ijj30pa09emy4.jpg)

```java
public class DexClassLoader extends BaseDexClassLoader {
   // dexPath: 需要加载的文件列表，文件可以是包含了 classes.dex 的 JAR/APK/ZIP，也可以直接使用 classes.dex 文件，多个文件用 “:” 分割
 // optimizedDirectory: 存放优化后的 dex，可以为空
 // librarySearchPath: 存放需要加载的 native 库的目录
 // parent: 父 ClassLoader  
  public DexClassLoader(String dexPath, String optimizedDirectory,
            String librarySearchPath, ClassLoader parent) {
        super(dexPath, null, librarySearchPath, parent);
    }
}
```

DexClassLoader用来加载jar、apk，其实还包括zip文件或者直接加载dex文件，它可以被用来执行未安装的代码或者未被应用加载过的代码，也就是我们修复过的代码。

注意：**Android 8.0之后，BaseClassLoader第二个参数为(optimizedDirectory)为null,所以DexClassLoader与PathClassLoader并无区别**



**从上面我们可以看到，它们都继承于BaseDexClassLoader，并且它们真正的实现行为都是调用的父类方法，所以我们来看一下BaseDexClassLoader.**

#### [BaseDexClassLoader](http://androidxref.com/9.0.0_r3/xref/libcore/dalvik/src/main/java/dalvik/system/BaseDexClassLoader.java)

```java
public class BaseDexClassLoader extends ClassLoader {

  private static volatile Reporter reporter = null;
	
  //核心关注点
   private final DexPathList pathList;

	BaseDexClassLoader 构造函数有四个参数，含义如下：

 // dexPath: 需要加载的文件列表，文件可以是包含了 classes.dex 的 JAR/APK/ZIP，也可以直接使用 classes.dex 文件，多个文件用 “:” 分割
 // optimizedDirectory: 存放优化后的 dex，可以为空
 // librarySearchPath: 存放需要加载的 native 库的目录
 // parent: 父 ClassLoader
   public BaseDexClassLoader(String dexPath, File optimizedDirectory,
           String librarySearchPath, ClassLoader parent) {
      	//classloader,dex路径,目录列表，内部文件夹
       this(dexPath, optimizedDirectory, librarySearchPath, parent, false);
   }


   public BaseDexClassLoader(String dexPath, File optimizedDirectory,
           String librarySearchPath, ClassLoader parent, boolean isTrusted) {
       super(parent);
       this.pathList = new DexPathList(this, dexPath, librarySearchPath, null, isTrusted);

       if (reporter != null) {
           reportClassLoaderChain();
       }
   }
  
  ...
 
    public BaseDexClassLoader(ByteBuffer[] dexFiles, ClassLoader parent) {
        // TODO We should support giving this a library search path maybe.
        super(parent);
        this.pathList = new DexPathList(this, dexFiles);
    }
	
  //核心方法
    @Override
protected Class<?> findClass(String name) throws ClassNotFoundException {
  			//异常处理
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
  			//这里也只是一个中转,关注点在 DexPathList
        Class c = pathList.findClass(name, suppressedExceptions);
        if (c == null) {
            ClassNotFoundException cnfe = new ClassNotFoundException(
                    "Didn't find class \"" + name + "\" on path: " + pathList);
            for (Throwable t : suppressedExceptions) {
                cnfe.addSuppressed(t);
            }
            throw cnfe;
        }
        return c;
    }

 
  ...
}


```

从上面我们可以发现，BaseDexClassLoader其实也不是主要处理的类，所以我们继续去查找 DexPathList.



#### [DexPathList](http://androidxref.com/9.0.0_r3/xref/libcore/dalvik/src/main/java/dalvik/system/DexPathList.java#makeInMemoryDexElements)

```java
final class DexPathList {
  //文件后缀
	private static final String DEX_SUFFIX = ".dex";
	private static final String zipSeparator = "!/";

** class definition context */
private final ClassLoader definingContext;

//内部类 Element
private Element[] dexElements;

public DexPathList(ClassLoader definingContext, String dexPath,
        String librarySearchPath, File optimizedDirectory) {
    this(definingContext, dexPath, librarySearchPath, optimizedDirectory, false);
}

DexPathList(ClassLoader definingContext, String dexPath,
        String librarySearchPath, File optimizedDirectory, boolean isTrusted) {
    if (definingContext == null) {
        throw new NullPointerException("definingContext == null");
    }

    if (dexPath == null) {
        throw new NullPointerException("dexPath == null");
    }

    if (optimizedDirectory != null) {
        if (!optimizedDirectory.exists())  {
            throw new IllegalArgumentException(
                    "optimizedDirectory doesn't exist: "
                    + optimizedDirectory);
        }

        if (!(optimizedDirectory.canRead()
                        && optimizedDirectory.canWrite())) {
            throw new IllegalArgumentException(
                    "optimizedDirectory not readable/writable: "
                    + optimizedDirectory);
        }
    }

    this.definingContext = definingContext;

    ArrayList<IOException> suppressedExceptions = new ArrayList<IOException>();
    // save dexPath for BaseDexClassLoader
  	//我们关注这个 makeDexElements 方法
    this.dexElements = makeDexElements(splitDexPath(dexPath), optimizedDirectory,
                                       suppressedExceptions, definingContext, isTrusted);
    this.nativeLibraryDirectories = splitPaths(librarySearchPath, false);
    this.systemNativeLibraryDirectories =
            splitPaths(System.getProperty("java.library.path"), true);
    List<File> allNativeLibraryDirectories = new ArrayList<>(nativeLibraryDirectories);
    allNativeLibraryDirectories.addAll(systemNativeLibraryDirectories);

    this.nativeLibraryPathElements = makePathElements(allNativeLibraryDirectories);

    if (suppressedExceptions.size() > 0) {
        this.dexElementsSuppressedExceptions =
            suppressedExceptions.toArray(new IOException[suppressedExceptions.size()]);
    } else {
        dexElementsSuppressedExceptions = null;
    }
}
  
  
  
static class Element {
	//dex文件为null时表示 jar/dex.jar文件
 private final File path;
	
 //android虚拟机文件在Android中的一个具体实现
 private final DexFile dexFile;

 private ClassPathURLStreamHandler urlHandler;
 private boolean initialized;

 /**
  * Element encapsulates a dex file. This may be a plain dex file (in which case dexZipPath
  * should be null), or a jar (in which case dexZipPath should denote the zip file).
  */
 public Element(DexFile dexFile, File dexZipPath) {
     this.dexFile = dexFile;
     this.path = dexZipPath;
 }

 public Element(DexFile dexFile) {
     this.dexFile = dexFile;
     this.path = null;
 }

 public Element(File path) {
   this.path = path;
   this.dexFile = null;
 }
	
 public Class<?> findClass(String name, ClassLoader definingContext,
               List<Throwable> suppressed) {
   					//核心点，DexFile
           return dexFile != null ? dexFile.loadClassBinaryName(name, definingContext, suppressed)
                   : null;
       }
  
 /**
  * Constructor for a bit of backwards compatibility. Some apps use reflection into
  * internal APIs. Warn, and emulate old behavior if we can. See b/33399341.
  *
  * @deprecated The Element class has been split. Use new Element constructors for
  *             classes and resources, and NativeLibraryElement for the library
  *             search path.
  */
 @Deprecated
 public Element(File dir, boolean isDirectory, File zip, DexFile dexFile) {
     System.err.println("Warning: Using deprecated Element constructor. Do not use internal"
             + " APIs, this constructor will be removed in the future.");
     if (dir != null && (zip != null || dexFile != null)) {
         throw new IllegalArgumentException("Using dir and zip|dexFile no longer"
                 + " supported.");
     }
     if (isDirectory && (zip != null || dexFile != null)) {
         throw new IllegalArgumentException("Unsupported argument combination.");
     }
     if (dir != null) {
         this.path = dir;
         this.dexFile = null;
     } else {
         this.path = zip;
         this.dexFile = dexFile;
     }
 }
  ...
}

  
  
 ...
//主要作用就是将 我们指定路径中所有文件转化为DexFile，同时存到Eelement数组中
//为什么要这样做？目的就是为了让findClass去实现
private static Element[] makeDexElements(List<File> files, File optimizedDirectory,
  List<IOException> suppressedExceptions, ClassLoader loader, boolean isTrusted) {
  Element[] elements = new Element[files.size()];
  int elementsPos = 0;
  //遍历所有文件
  for (File file : files) {
      if (file.isDirectory()) {
         	//如果存在文件夹，查找文件夹内部查询
          elements[elementsPos++] = new Element(file);
        //如果是文件
      } else if (file.isFile()) {
          String name = file.getName();
          DexFile dex = null;
        //判断是否是dex文件
          if (name.endsWith(DEX_SUFFIX)) {
              // Raw dex file (not inside a zip/jar).
              try {
                	//创建一个DexFile
                  dex = loadDexFile(file, optimizedDirectory, loader, elements);
                  if (dex != null) {
                      elements[elementsPos++] = new Element(dex, null);
                  }
              } catch (IOException suppressed) {
                  System.logE("Unable to load dex file: " + file, suppressed);
                  suppressedExceptions.add(suppressed);
              }
          } else {
              try {
                  dex = loadDexFile(file, optimizedDirectory, loader, elements);
              } catch (IOException suppressed) {
                  /*
                   * IOException might get thrown "legitimately" by the DexFile constructor if
                   * the zip file turns out to be resource-only (that is, no classes.dex file
                   * in it).
                   * Let dex == null and hang on to the exception to add to the tea-leaves for
                   * when findClass returns null.
                   */
                  suppressedExceptions.add(suppressed);
              }

              if (dex == null) {
                  elements[elementsPos++] = new Element(file);
              } else {
                  elements[elementsPos++] = new Element(dex, file);
              }
          }
          if (dex != null && isTrusted) {
            dex.setTrusted();
          }
      } else {
          System.logW("ClassLoader referenced unknown path: " + file);
      }
  }
  if (elementsPos != elements.length) {
      elements = Arrays.copyOf(elements, elementsPos);
  }
  return elements;
}
  
  ---
 private static DexFile loadDexFile(File file, File optimizedDirectory, ClassLoader loader,
Element[] elements)throws IOException {
     //判断可复制文件夹是否为null
     if (optimizedDirectory == null) {
         return new DexFile(file, loader, elements);
     } else {
       	//如果不为null，则进行解压后再创建
         String optimizedPath = optimizedPathFor(file, optimizedDirectory);
         return DexFile.loadDex(file.getPath(), optimizedPath, 0, loader, elements);
     }
 }
  
  -----
public Class<?> findClass(String name, List<Throwable> suppressed) {
    //遍历初始化好的DexFile数组,并由Element调用 findClass方法去生成
    for (Element element : dexElements) {
      	//
        Class<?> clazz = element.findClass(name, definingContext, suppressed);
        if (clazz != null) {
            return clazz;
        }
    }

    if (dexElementsSuppressedExceptions != null) {
        suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
    }
    return null;
}
```

上面的代码有点复杂，我摘取了其中一部分我们需要关注的点，便于我们进行分析：

在**BaseDexClassLoader**中，我们发现最终加载类的是由 **DexPathList** 来进行的，所以我们进入了 **DexPathList** 这个类中，我们可以发现 在初始化的时候，有一个关键方法需要我们注意 **makeDexElements**。而这个方法的主要作用就是将 我们指定路径中所有文件转化为 **DexFile** ，同时存到 **Eelement** 数组中。

而最开始调用的 **DexPathList中的findClass()** 反而是由**Element** 调用的 **findClass**方法，而**Emement**的**findClass**方法中实际上又是 **DexFile** 调用的 **loadClassBinaryName** 方法，所以带着这个疑问，我们进入 DexFile这个类一查究竟。



#### [DexFile](http://androidxref.com/9.0.0_r3/xref/libcore/dalvik/src/main/java/dalvik/system/DexFile.java)

```java
public final class DexFile {
*
 If close is called, mCookie becomes null but the internal cookie is preserved if the close
 failed so that we can free resources in the finalizer.
/
@ReachabilitySensitive
private Object mCookie;

private Object mInternalCookie;
private final String mFileName;
...
DxFile(String fileName, ClassLoader loader, DexPathList.Element[] elements) throws IOException {
     mCookie = openDexFile(fileName, null, 0, loader, elements);
     mInternalCookie = mCookie;
     mFileName = fileName;
     //System.out.println("DEX FILE cookie is " + mCookie + " fileName=" + fileName);
 }
  
 //关注点在这里
public Class loadClassBinaryName(String name, ClassLoader loader, List<Throwable> suppressed) {
     return defineClass(name, loader, mCookie, this, suppressed);
 }

//
 private static Class defineClass(String name, ClassLoader loader, Object cookie,
                                  DexFile dexFile, List<Throwable> suppressed) {
     Class result = null;
     try {
       //这里调用了一个 JNI层方法
         result = defineClassNative(name, loader, cookie, dexFile);
     } catch (NoClassDefFoundError e) {
         if (suppressed != null) {
             suppressed.add(e);
         }
     } catch (ClassNotFoundException e) {
         if (suppressed != null) {
             suppressed.add(e);
         }
     }
     return result;
 }

  private static native Class defineClassNative(String name, ClassLoader loader, Object cookie,
                                               DexFile dexFile)
         throws ClassNotFoundException, NoClassDefFoundError;
```

我们从 **loadClassBinaryName** 方法中发现，调用了 **defineClass** 方法，最终又调用了 ***defineClassNative*** 方法，而 **defineClassNative** 方法是一个JNI层的方法，所以我们无法得知具体如何。但是我们思考一下，从开始的 **BaseDexClassLoader**一直到现在的 **DexFile**,我们一直从入口找到了最底下，不难猜测，这个 **defineClassNative** 方法内部就是 C/C++帮助我们以字节码或者别的生成我们需要的 dex文件，这也是最难的地方所在。

最后我们再用一张图来总结一下Android 中类加载的过程。

![image-20191125161551841](https://tva1.sinaimg.cn/large/006y8mN6ly1g9acumt6kgj30k40f4dgs.jpg)



在了解完上面的知识之后，我们来总结一下，Android中热修复的原理？

> ***Android中既然已经有了DexClassLoader和 PathClassLoader，那么我在加载过程中直接替换我自己的Dex文件不就可以了，也就是先加载我自己的Dex文件不就行了,这样不就实现了热修复。***

#### 真的这么简单吗？热修复的难点是什么？

- 资源修复
- 代码修复
- so库修复

抱着这个问题，如何选用一个最合适的框架，是我们Android开发者必须要考虑的，下面我们就分析一下各方案的差别。



### 如何选择热修复框架？

目前市场上的热修复框架很多，从阿里热修复网站找了一个图来对比一下：

| 平台       |    Sophix    |    AndFix    |     Tinker     |     Qzone      |     Robust     |
| :--------- | :----------: | :----------: | :------------: | :------------: | :------------: |
| 即时生效   |     yes      |     yes      |       no       |       no       |      yes       |
| 性能损耗   |     较小     |     较小     |      较大      |      较大      |      较小      |
| 侵入式打包 | 无侵入式打包 | 无侵入式打包 | 依赖侵入式打包 | 依赖侵入式打包 | 依赖侵入式打包 |
| Rom体积    |     较小     |     较小     |      较大      |      较小      |      较小      |
| 接入复杂度 |  傻瓜式接入  |   比较简单   |      复杂      |    比较简单    |      复杂      |
| 补丁包大小 |     较小     |     较小     |      较小      |      较大      |      一般      |
| 全平台支持 |     yes      |     yes      |      yes       |      yes       |      yes       |
| 类替换     |     yes      |     yes      |      yes       |      yes       |       no       |
| so替换     |     yes      |      no      |      yes       |       no       |       no       |
| 资源替换   |     yes      |      no      |      yes       |      yes       |       no       |

简单划分就是3大巨头，阿里，腾讯，美团。并不是谁支持的功能多就用谁，在接入方面我们需要综合考虑。

详细的技术对比请参考 [Android热修复技术选型——三大流派解析](https://mp.weixin.qq.com/s/uY5N_PSny7_CHOgUA99UjA?spm=a2c4g.11186623.2.13.395a3a3cJMDbyf)

以我个人的体验来说吧：目前体验了Tinker和 Sophix

**Tinker**

> Tinker的集成有点麻烦，我个人觉得挺简单，而且补丁管理系统 TinkerPatch是收费的(有免费额度)，补丁下发慢,大概需要5分钟的等待时间。
>
> Tinker有一个免费版后台,Bugly,补丁管理是免费的，热修复用的Tinker，集成很那啥。。。em，建议多读官网教程看视频，因为有补丁上传监测，下发一个补丁需要5-10分钟等待生效，撤回补丁需要10分钟左右生效，而且一次可能不会生效，后台观察日志需要多次才可以实现补丁撤回。(测试设备：小米5s Plus，Android 8.0)
>
> 最后总结：
>
> 优点：免费，简单
>
> 缺点：集成麻烦，出现问题无法第一时间得到解决方案，毕竟免费的理解一下
>
> 性能方法：需要冷启动之后才会生效

**Sophix**

> 官网教程详细，完全傻瓜式，响应快，出现问题，解决效率高，毕竟花了钱的。
>
> 性能方面：冷启动+即时响应(有条件)，
>
> 有点：功能最多，支持版本最多，解决问题快
>
> 缺点：付费

别的框架没有体验，也就不妄自评价了。关于以上方案的实现原理，大家可以点击[Android热修复技术选型——三大流派解析](https://mp.weixin.qq.com/s/uY5N_PSny7_CHOgUA99UjA?spm=a2c4g.11186623.2.13.395a3a3cJMDbyf)，或者百度搜索。简单了解并不困难。





### 热修复的注意事项

有了热修复，我们就可以为所欲为了吗？

**开始讲骚话**：

> 并不是，热修复受限于各种机型设备，而且也有失败的可能性，所以我们开发者，对于补丁包同样也要抱有敬畏之心。
>
> *对于热修复同样也由于严格的过程，但是我们日常开发至少要保证以下几点：*
>
> **debug-> 打补丁包->开发设备测试->灰度下发(条件下发)->全量下发**



**下面针对我开发中遇到的问题，给出解决方案。**

### 热修复与多渠道

多渠道打包使用 美团 的一键打包方案。补丁包的话，其实并不会影响，因为补丁包一般改动的代码相同，但前提是需要保证我们每个渠道基准包没问题。如果改动代码有区别，那就需要针对这个渠道单独打补了。



### 自动化构建与热修复

Android开发一般集成了 Jenkins 或者别的自动化打包工具，我们一般基准包都在 app/build/bakApk目录下，所以我们可以通过编写 shell 命令，在jenkins中打包时，将生成的基准包移动到一个特定的文件夹即可。**tinker**,**Sophix**都是支持服务器后台的，所以我们也可以通过自动化构建工具上传补丁包，如果相应的热修复框架不支持服务器管理的话，那么可以将补丁包上传的指定的文件夹，然后我们app打开时，访问我们的服务器接口下拉最新的补丁包，然后在service中合成。不过 **Tinker(bugly) **, **Sophix** 都是支持后台管理，所以具体使用那种方案我们自行选择。



***关于热修复的到这里就基本写完了，散散落落居然写了这么多，其实难的不是热修复，而是Android中类加载的过程及一些基础相关知识，理解了这些，我们才能真正明白那些优秀的框架到底是怎样去修复的。***

如果本文有帮到你的地方，不胜荣幸。如果有什么地方有错误或者疑问，也欢迎大家提出。

# 