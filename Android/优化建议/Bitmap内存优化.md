# Bitmap内存优化

## 为什么Bitmap需要高效加载？

在日常开发中，我们不免会使用到Bitmap,而bitmap确实实在在的是内存使用的 “大户”，如何更好的使用 bitmap,减少其对 App内存的使用，是我们开发中不可回避的问题。

为了解决这个问题，就出现了Bitmap 的高效加载策略。其实核心思想很简单。假设通过InmageView 来显示图片，很多时候 ImageVIew并没有原始图片的尺寸那么大，这个时候把整个图片加载进来再设置ImageView,显示是没有必要的，因为ImageView根本没办法显示原始图片。这时候就可以按一定的采样率来将图片缩小后在加载进来，这样图片既能在ImageView显示出来，又能降低内存占用从而在一定程度上避免OOM，提高了Bitmap加载时的性能。

## 基础了解

**我们先了解一下，Bitmap到底占用多大的内存。**

Bitmap作为位图，需要读入一张图片每一个像素点的数据，其主要占用内存的地方也正是这些像素数据。对于像素数据总大小，我们可以猜想为：像素总数量 x 每个像素的字节大小，而像素总数量在矩形屏幕的表现下，应该是：横向像素数量 x 纵向像素数量，结合得到：

> Bitmap内存占用 = 像素数据总大小=横向像素数量 x 纵向像素数量 x 每个像素的字节大小

### 单个像素的字节大小

单个像素的字节大小由Bitmap 的一个可配置参数 Config 来决定。

Bitmap 中，存在一个 枚举类 Config,定义了Android 中支持的 Bitmap配置。

| Config        | 占用字节大小（byte） | 说明                                                       |
| ------------- | -------------------- | ---------------------------------------------------------- |
| ALPHA_8 (1)   | 1                    | 单透明通道                                                 |
| RGB_565 (3)   | 2                    | 简易RGB色调                                                |
| ARGB_4444 (4) | 4                    | 已废弃                                                     |
| ARGB_8888 (5) | 4                    | 24位真彩色                                                 |
| RGBA_F16 (6)  | 8                    | Android 8.0 新增（更丰富的色彩表现HDR）                    |
| HARDWARE (7)  | Special              | Android 8.0 新增 （Bitmap直接存储在graphic memory）**注1** |

而Bitmap默认是使用24位真彩色加载的。

```java
  * Image are loaded with the {@link Bitmap.Config#ARGB_8888} config by default.
  */
  public Bitmap.Config inPreferredConfig = Bitmap.Config.ARGB_8888;
```



## Bitmap高效加载的具体方式

### 加载Bitamp的方式

bitmap在Android中指的是一张图片。通过BitmapFactory类提供的4类方法：

decodeFile,decodeResouce,decodeStream和 decodeByteArray,分别从文件系统，资源，输入流和字节数组中加载出一个 Bitmap 对象，其中decodeFiled,decodeResource又间接调用了 decodeStream 方法，这4类 方法最终是在Android的底层实现的，对应着BitmapFactory类的几个native方法。

### BitmapFactory.Options的参数

#### inSampleSize参数(采样率)

上述4类方法都支持BitmapFactory.Options参数，而Bitmap的按一定采样率进行缩放就是通过 BitmapFactory.Options参数实现的，主要用到了 inSampleSize参数，即采样率。通过对 inSampleSize 的设置，对图片的像素的高和款进行缩放。

当 inSampleSize=1 ,即采样后的图片大小为图片的原始大小，小于1，也按照1来计算。当 inSampleSize>1,即采样后的拖欠将会缩小，缩放比例为1/(inSampleSize的二次方)。

> 例如：一张 1024—1024像素的图片，采用ARG8888 格式存储，那么内存大小1024x1024x4=4m.如果 inSampleSize=2,即采样后图片内存大小为 512x512X4=1m

**注意：官方文档中指出，inSampleSize的取值应该总是2的指数，如1，2，4，8等。如果外界传入的 inSampleSize的值不为2的指数，那么系统会向下取整并选择成立一个最接近2的指数来代替。比如3，系统会选择2来代替。不过并非在所有的Android版本都成立**

**关于 inSampleSize 取值的注意事项：**通常是根据图片宽高实际的大小/需要的宽高大小，分别计算出宽和高的缩放比。单应该取其中最小的缩放比，避免缩放图片大小，到达指定控件中不能铺满，需要拉伸从而导致模糊。

例如：ImageView的大小是 100x100 像素，而图片的原始大小是 200x300，那么宽的缩放比是 2，高的缩放比是 3，如果最终 inSampleSize=2,那么缩放后的图片大小 100x150，仍然合适 ImageView。如果inSamleSize=3,那么缩放后的图片大小小于 ImageView所期望的大小。这样图片就会被拉伸而导致模糊。



#### inJustDecodeBounds 参数

我们需要获取加载的图片的宽高信息，然后交给inSampleSize 参数选择缩放比缩放，那么如何能不先加载图片却能获取得图片的宽高信息，通过 inJustDecodeBunds=true,然后加载图片就可以实现只解析图片的宽高信息，并不会真正的加载图片，所以这个操作是轻量级的。当获取了宽高信息，计算出缩放比后，然后在将 inJustDecodeBounds=false,再重新加载图片，就可以加载缩放后的图片。

**注意：BitmapFactory 获取得图片宽高信息和图片的位置以及程序运行的设备有关，比如同一张图片放在不同的drawable目录下或者程序运行在不同屏幕密度的设备上，都可能导致BitmapFactory 获取到不同的结果，和 Android 的资源加载机制有关。**



### 高效加载Bitmap的流程

1. 将BitmapFactory.Options的 inJustDecodeBounds 参数设置为true并加载图片。
2. 从BitmapFactory.Options中取出图片的原始宽高信息，他们对应于uouytWidth 和 outHeight参数。
3. 根据采样率的规则并结合目标View 的所需大小计算出采样率 inSampleSize.
4. 将BitmapFactory.Options 的inJustDecodeBounds 参数设为 false,然后重新加载图片。

代码操作：

```java
private Bitmap showBit(Resources resources, int id, int width, int height) {
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    Bitmap bitmap;
    //加载图片
    BitmapFactory.decodeResource(resources, id, options);
    Log.e("demo","w"+options.outWidth+"----h"+options.outHeight);
    //计算缩放比
    options.inSampleSize = calculateInsamplSize(options, width, height);
    //重新加载图片
    options.inJustDecodeBounds = false;
    bitmap = BitmapFactory.decodeResource(resources, id, options);
    Log.e("demo",bitmap.getWidth()
            +"------height"+bitmap.getHeight()+"------size"
            +options.inSampleSize+"-----byte"+bitmap.getRowBytes());
    return bitmap;
}

private int calculateInsamplSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    int height = options.outHeight;
    int width = options.outWidth;
    int sizeSimple = 1;
    if (height > reqHeight && width > reqWidth) {
        int sizew=width/reqWidth;
        int sizeh=height/reqHeight;
        //选其中最小的边为标准
        int sizemode=sizew>sizeh?sizeh:sizew;
        if (sizemode>sizeSimple){
            return sizeSimple*sizemode;
        }
    }
    return sizeSimple;
}
```

观察打印数据：

![image-20190908195328288](https://tva1.sinaimg.cn/large/006y8mN6ly1g6sct1untzj31p805cta5.jpg)

经过我们压缩之后，其图片大小占用1260字节，分辨率也是随之下降，不过都在我们所设定的范围之内，下面我们看看，如果不压缩，结果是怎么样。

更改inSampleSize=1，也就是默认原图显示。效果如下：

![image-20190908195643081](https://tva1.sinaimg.cn/large/006y8mN6ly1g6sd0ap7xrj31kg05ojt1.jpg)