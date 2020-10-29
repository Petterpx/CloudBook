# Flutter-资源管理



![image-20201026131503372](https://tva1.sinaimg.cn/large/0081Kckwgy1gk2nz5bklkj30l806t3yj.jpg)





Flutter安装包中会包含代码和assets (资源)两部分，其中 assets 是会打包到程序安装包中，可以运行时访问，常见的 assets 类型包括静态数据(json文件)，配置文件，图标和图片等。



## 如何指定assets

Flutter使用 **pusbspec.yaml** 来管理程序所需资源，对于每一个资源文件，都需要在 pushspec 中声明，否则调用时就会出现找不到资源文件的报错。

**assets** 指定应包含在应用程序中的文件，每个asset 都通过相对于 **pushspec.yaml** 文件所在的文件系统路径来标识自身路径，不过 assets 的声明顺序无关紧要，你可以放到任意文件夹下，如下图所示：

<center class="half" >
    <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gk1g4jar0xj309l08mdgd.jpg" width="400"/><img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk2g5jtwxjj30m6088ab0.jpg" width="450"/>
</center>








## Asset变体(variant)

构建过程支持 “ asset变体 ”的概念，不同版本的 asset 可能会显示在不同的上下文中。在 **pubspec.yml** 的 assets 部分指定assets 路径时，构建过程中，会在相邻子目录中查找具有相同名称的任何文件。这些文件随后会与指定的 assets一起被包含在 asset bundle中。

例如：如果应用程序目录中有以下文件：

```yaml
../images/icon.png
../images/dark/icon.png
```

在你的 **pubspec.yml** 文件中只需包含

```yaml
flutter:
   assets:
     - images/icon.png
```

在实际构建过程中， 上面两个文件都将打入你的 asset bundle 中，前者被认为是 main asset(主资源),后者被认为是一种变体(variant)。

在选择匹配当前设备分辨率的图片是，Flutter 会使用 asset 变体，在以后，可能会将这种机制扩展到本地化，阅读提示等方面。



## 加载 assets

通过 **AssetBundle** 对象访问其asset,有两种主要方法允许从 Asset bundle中加载字符串或图片(二进制)文件。

### 加载图片

在不同的分辨率设备上，**AssetImage** 可以选择不同分辨率的图片进行显示，但为了让 Flutter 能知道如何去寻找，对于图片的位置，必须按照特定的目录结构，如下：

```dart
../icon.png
../2.0x/icon.png
../3.0x/icon.png
```

>  **注意**：在pubspec 声明时只需 声明 **xx/icon.png** 即可，其中xx是你具体的包位置。如果没懂，请上滑查看 Asset变体

当这样放置图片之后，比如在分辨率为2.7的设备上，3.0x的图片将被选择。

> **注意**：如果未在 **Image** widget上指定渲染图像的宽高和宽度，那么 Image widget将占用与主资源相同的屏幕空间大小，比如主资源也就是默认的 icon.png大小是 100 x 100px,2.0x的是 300 x 300px,那么实际在渲染时，这张图将被渲染为 100 x 100px。

**pubspec.yaml** 中 asset 部分中的每一项都应该与实际文件相同，但主资源项除外。当主资源缺少某个资源时，会按分辨率从低到高的顺序去选择，也就是说 1x 中没有就去2x去找，2x没有就去3x找。

**示例代码如下：**

```yaml
flutter:
  uses-material-design: true
  assets:
      - images/icon.jpeg
```

```dart
Image(width: 200, height: 200, image: AssetImage("images/icon.jpeg"))
```

**注意**

这里我们使用的是 **AssetImage**,其并非是一个 widget,实际上是一个 **ImageProvider** ,如果你可能期望直接得到一个现实图片的 widget,那么可以使用 Image.asset()，如下：

```dart
Image.asset("images/icon.jpeg");
```

> 使用默认的 asset bundle 加载资源时，内部会自动处理分辨率等，这些处理对于开发者来说无感知。当然如果使用一些更低级别的类，如 **ImageStream** 或 **ImageCache** 时就会有其他相关参数，如缩放。
>

### 加载文本

```yaml
flutter:
  uses-material-design: true
  assets:
      - data/test.json
```

```dart
rootBundle.loadString("data/test.json");
```

### 示例动画

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1gk2iio1qlwg30by0mw7wh.gif" alt="读图读文本" style="zoom:50%;" />



## 加载依赖包中的资源图片

如果要加载某个依赖包中的图像，必须给 **AssetImage** 提供 **package**参数。

比如我们依赖了一个名为 test_icons 的包，它的结构如下：

```dart
.../images/icon.png
.../images/2.0x/icon.png
.../images/3.0x/icon.png
```

在我们加载图像时，就要使用如下两种方式(显示声明package)：

```dart
AssetImage("images/icon.png",package:"test_icons")
```

```dart
Image.asset("images/icon.png",package:"test_icons")
```



在加载时，我们也可以选择实际在依赖包中存在，但未在其 **pubspec.yaml** 中声明的图片，对于这种情况，必须在 pubspec.yaml 中指定包含哪些图像。例如，名为 **test_icons**包中，包含以下图片：

```
.../images/test.png
.../images/background_gray.png
```

如果我们要使用其中的 test.png ,就必须在 **pubspec.yaml** 的 assets 部分中显示声明：

```yaml
flutter:
  uses-material-design: true
  assets:
      - packages/test_icons/images/icon.jpeg
```





## 参考资料

Flutter实战-书籍

