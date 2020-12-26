# Flutter | 常见的一些基础组件

## Text

### 基础使用

Text 用于显示简单的样式文本，其中包含了一些控制文本显示样式的属性，比如：

|                             示例                             |                             代码                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201206223018812](https://tva1.sinaimg.cn/large/0081Kckwly1gleiaeyanvj30af091abe.jpg) | ![image-20201206222950684](https://tva1.sinaimg.cn/large/0081Kckwly1gleia2ccqoj30t00cajsj.jpg) |



### TextStyle

TextStytle 用于指定文本显示的样式如颜色，字体，粗细，背景等。

|                             示例                             |                             代码                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201206225022168](https://tva1.sinaimg.cn/large/0081Kckwly1gleivf4k0pj30av0990so.jpg) | ![image-20201206224953601](https://tva1.sinaimg.cn/large/0081Kckwly1gleiusn0khj30l10eign2.jpg) |

需要注意的是：

- height : 该属性用于指定行高，其只是一个因子，具体的行高等于 fontSize * height.
- fontFamily : 由于不同平台默认支持的字体集不同，所以在手动指定字体时一定要先在不同平台测试一下。
- fontSize : 该属性和Text 的 textScaleFactor 都用于控制字体大小。但是有两个主要区别；
  - fontSize 可以精确指定字体大小，而 **textScaleFactor** 只能通过缩放比例来控制
  - **textScaleFactor** 主要是用于系统字体大小设置改变时对Flutter 应用字体进行全局调整，而 fontSize 通常用于单个文本，字体大小不会跟随系统字体大小变化。

### TextSpan

在上面例子中，我们的内容只能按同一种样式，如果我们需要对一个Text内容的不同步按照不同的样式显示，这时就可以使用 TextSpan ，它代表文本的一个片段，我们看看 TextSpan 的定义：

![image-20201206232549264](https://tva1.sinaimg.cn/large/0081Kckwly1glejw6dq0fj315a0ik416.jpg)



### DefaultTextStyle

在Widget 树中，文本的样式默认是可以被继承的(子类文本类组件未指定具体样式时可以使用Widget树中父级设置的默认样式)，因此，如果我们在 Widget 树的某一个节点处设置一个默认的文本样式，那么该节点的子树中所有文本都会默认使用这个样式，而 **DefultTextStyle** 正是用于设置默认文本样式：

|                             图示                             |                             代码                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201207222258287](https://tva1.sinaimg.cn/large/0081Kckwly1glfnp4f73yj305b05a74b.jpg) | ![image-20201207222324182](https://tva1.sinaimg.cn/large/0081Kckwly1glfnpj199qj30j80ffwga.jpg) |



### 字体

对于更换我们显示的字体，常见的客户端开发同学都不会陌生，而Flutter也不用自然也支持这个功能。

在Flutter中使用字体一般分为两步。

1. 在 pubspec.yaml 中声明；
2. 通过 TextStyle 属性使用；

#### 在 asset 中声明

要将字体打包到我们应用中，和使用其他资源一样，要先在 pubspec.yaml 中声明，然后将我们字体复制到 pubspec.yaml 中指定的位置。示例如下：

**配置过程**

![image-20201207234225918](https://tva1.sinaimg.cn/large/0081Kckwly1glfpzrpmyyj30x40a976s.jpg)

[Google-Font:字体下载网站](https://fonts.google.com/)

**示例如下：**

![image-20201207233245423](https://tva1.sinaimg.cn/large/0081Kckwly1glfpposrrej30qg07dtbf.jpg)

> **注意：**如果使用后发现没有效果，确认代码没问题情况下，记得先卸载app,再运行即可看到效果。😂



## 按钮

### RaisedButton





## 图片

在Flutter中，我们可以通过 **Image** 组件来加载并显示图片，Image 的数据源可以是 asset,文件，内存以及网络

### ImageProvider

**ImageProvider** 是一个抽象类，主要是定义了图片数据获取的接口 **load()** ,从不同的数据源获取图片需要实现不同的 **ImageProvider** ,比如 AssetImage 是实现了从 Asset 中加载图片的 ImageProvider ,而 NetWorkImage 实现了从网络加载图片的 ImageProvider.



### Image

### 加载本地图片

```dart
Image(
  image: AssetImage("images/jenkins.jpg"),
  width: 100,
)

//快捷方式
Image.asset()
```

### 加载网络图片

```dart
Image(
  image: NetworkImage(
      "https://dss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=3942751454,1089199356&fm=26&gp=0.jpg"),
  width: 100,
)

//快捷方式       
Image.network()
```

### 参数

Image 同样也定义了一系列参数，通过这些参数可以控制图片的显示外观，大小，混合效果等。如下所示:

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1glml4kgv28j30d20fqjta.jpg" alt="image-20201213221258405" style="zoom: 67%;" />

其中width和height，如果用户没有指定，则会根据当前父容器的规则，尽可能显示其原始大小，如果只设置了其中一个，那么另一个属性默认会按比例缩放，但我们也可以通过 fit 属性来指定其适应规则。类似于Android开发中的 **scaleType**.

Fit : 用于在图片显示空间和图片本身大小不同时指定图片的适应模式，适应模式是在 **BoxFit** 中定义，有如下值。

- fit 拉伸填充显示空间，图片本身长宽会发生变化，图片会变形
- coveran 图片的长宽比放大后居中填满显示空间，图片不会变形，超出显示的部分会被裁剪
- contain  图片的默认适应规则，图片会在保证图片本身长宽比不变的情况下缩放以适应当前显示空间，图片不会变形
- fitWidth 图片的宽度会缩放到显示空间的宽度，高度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被裁剪
- fitHeight 图片的高度会缩放到显示空间的高度，宽度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被裁剪
- none : 图片没有适应策略，会在显示空间内显示图片，如果图片比显示空间大，则显示空间只会显示图片中间部分。

具体示例如下。

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![image-20201214004459865](https://tva1.sinaimg.cn/large/0081Kckwly1glmpipsoglj30990j3djq.jpg) |

如图所示，我对每个图片设置了相应的大小，然后在不同的 fit 下，Image 会自动进行相应的调整。

#### color 和 colorBlendMode 

用于在图片绘制时可以对每一个像素进行颜色混合处理， **color** 指定混合色，而 colorBlendMode 指定混合模式，如下：

|                             代码                             |                             图示                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201214231718058](https://tva1.sinaimg.cn/large/0081Kckwly1glnslrtb47j30d706zmxq.jpg) | ![image-20201214231603993](https://tva1.sinaimg.cn/large/0081Kckwly1glnsluztcij307r03a3yc.jpg) |



#### repeat

当图片本身大小小于显示空间时，指定图片的重复规则，如下：

|                             代码                             |                             图示                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201214231812758](https://tva1.sinaimg.cn/large/0081Kckwly1glnsmpy5u7j30bm06y3z0.jpg) | ![image-20201214231831642](https://tva1.sinaimg.cn/large/0081Kckwly1glnsn1ru9yj305204874j.jpg) |



### 注意

Flutter对加载过的图片默认都存在缓存，即缓存在(内存)中，其默认最大缓存数量是1000，最大缓存空间为100m。



### Icon

在Flutter中，我们也可以使用 iconfont，iconfont即 **字体图标**，在以往的Android开发中，我们也会接触到，它是将图标做成字体文件，然后通知指定不同的字符而现实不同的图片。

> 在字体文件中，每一个字符都对应一个位码，而每一个位码对应一个现实字形，不同的字体就是指字形不同，及字符对应的字形是不同的，在 iconFont 中，只是将位码对应的 字形做成了图标，所以不同的字符最终就会渲染成图标。

iconfont 相比直接使用图片具备如下优势：

1. 体积小
2. 矢量图标，不会因为放大缩小而影响显示清晰度
3. 可应用于文本样式，像文本一样改变字体图标颜色，大小对齐等
4. 可以通过 TextSpan 与文本实现混用

#### 使用Material Design 字体图标

Flutter 默认包含了一套 Material Design 字体图标， 使用方式如下：

![image-20201214233056203](https://tva1.sinaimg.cn/large/0081Kckwly1glnszyq8q9j30qt072q46.jpg)

```
uses-material-design: true
```

当然，这个默认就是打开的 👏 。其相关图标地址如下：https://material.io/tools/icons/

#### 使用自定义IconFont

如下使用[阿里矢量图](https://www.iconfont.cn/?spm=a313x.7781069.1998910419.d4d0a486a)。

![image-20201214235756447](https://tva1.sinaimg.cn/large/0081Kckwly1glnts28p58j30ph07j3zx.jpg)

```dart
class AliIcons {
  static const IconData cat =
      const IconData(0xef6a, fontFamily: "AliIcon", matchTextDirection: true);
}
```

|                             代码                             |                             图示                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201214235933283](https://tva1.sinaimg.cn/large/0081Kckwly1glnttqbtwzj30c303cweq.jpg) | ![image-20201214235909579](https://tva1.sinaimg.cn/large/0081Kckwly1glnttwhry6j305p02lwe9.jpg) |

关于[Flutter中阿里矢量图使用教程](https://juejin.cn/post/6844903813485166600)。



## 单选框和复选框todo



## 输入框和表单

## TextField

```dart
TextField(
  //自动获取焦点
  autofocus: true,
  //设置输入框默认键盘类型
  keyboardType: TextInputType.number,
  // TODO: 2020/12/17 方式1：通过onChanged自己记录
  onChanged: (value) {
    print("通过onChanged监听--$value");
  },

  decoration: InputDecoration(
      labelText: "输入名字",
      hintText: "我是一个提示",
      prefixIcon: Icon(Icons.verified_user)),
)
```

### 常用参数

- Controller :

  输入框的控制器，通过其可以设置获取编辑框的内容，选择编辑内容，监听编辑文本改变事件。一般情况下都需要显示提供一个 **controller** 来于文本进行交互。如果用户没有自己提供，则 TextField 内部则会自己创建一个。

- focusNode :

  用于控制 TextField 是否占有当前键盘的输入焦点，是我们用户与键盘交互的一个句柄 (handle)

- IntPutDecoration:

  控制TextField 的外观显示，比如文本，背景颜色，边框等

- keyboardType: 

  设置该输入框默认的键盘输入类型，相应的范围如下：

| TextInputType枚举值 | 解释                                                   |
| ------------------- | ------------------------------------------------------ |
| text                | 文本输入框                                             |
| muitline            | 多行文本，需要和 maxLines配合去使用，(设为null或大于1) |
| number              | 数字，设置后会弹出数字键盘                             |
| phone               | 优化后的电话号码输入键盘；会弹出数字键盘并显示 “ # ”   |
| datetime            | 优化后的日期输入键盘，Android上会显示":-"              |
| emailAddress        | 优化后的电子邮件地址；会显示”@“                        |
| Url                 | 优化后的url输入键盘；会显示"/."                        |

- **textInputAction**：键盘动作按钮图标，即我们常见的回车图标。
- **style**：正在编辑的文本样式
- `textAlign`: 输入框内编辑文本在水平方向的对齐方式。
- `autofocus`: 是否自动获取焦点。
- `obscureText`：是否隐藏正在编辑的文本，如用于输入密码的场景等，文本内容会用“•”替换。
- `maxLines`：输入框的最大行数，默认为1；如果为`null`，则无行数限制。
- `maxLength`和`maxLengthEnforced` ：`maxLength`代表输入框文本的最大长度，设置后输入框右下角会显示输入的文本计数。`maxLengthEnforced`决定当输入文本长度超过`maxLength`时是否阻止输入，为`true`时会阻止输入，为`false`时不会阻止输入但输入框会变红。
- `onChange`：输入框内容改变时的回调函数；注：内容改变事件也可以通过`controller`来监听。
- `onEditingComplete`和`onSubmitted`：这两个回调都是在输入框输入完成时触发，比如按了键盘的完成键（对号图标）或搜索键（🔍图标）。不同的是两个回调签名不同，`onSubmitted`回调是`ValueChanged<String>`类型，它接收当前输入内容做为参数，而`onEditingComplete`不接收参数。
- `inputFormatters`：用于指定输入格式；当用户输入内容改变时，会根据指定的格式来校验。
- `enable`：如果为`false`，则输入框会被禁用，禁用状态不接收输入和事件，同时显示禁用态样式（在其`decoration`中定义）。
- `cursorWidth`、`cursorRadius`和`cursorColor`：这三个属性是用于自定义输入框光标宽度、圆角和颜色的。

### 焦点控制

对于TextField 的焦点，我们可以通过 FocusNode 和 FocusScopeNode 来控制，默认情况下，焦点由 FocusScope 来管理，其代表焦点控制范围，在这个范围内，我们可以用个 FocusScopeNode 在输入框之间移动焦点，设置默认焦点等。

如下示例：

```dart
FocusNode focusNode1 = FocusNode();
FocusNode focusNode2 = FocusNode(); 
FocusScopeNode? _focusScopeNode;

TextField(
            focusNode: focusNode1,
            //用于控制TextField外观显示，如文本，背景色，边框等
            decoration: InputDecoration(
                labelText: "输入名字",
                hintText: "我是一个提示",
                prefixIcon: Icon(Icons.verified_user)),
          ),
          TextField(
            focusNode: focusNode2,
            decoration: InputDecoration(
                labelText: "我是测试2",
                hintText: "我是第二个测试",
                prefixIcon: Icon(Icons.access_time)),
          ),
          RaisedButton(
            // TODO: 2020/12/17 移动焦点
            onPressed: mobileFocus,
            child: Text("移动焦点"),
          )
            
            
 /// 移动焦点 */
  mobileFocus() {
    //方式1
    FocusScope.of(context).requestFocus(focusNode2);

    // 方式2
    // if (_focusScopeNode == null) {
    //   _focusScopeNode = FocusScope.of(context);
    // }
    // _focusScopeNode?.requestFocus(focusNode2);
  }
```



### 监听焦点状态改变事件

通过 FocusNode 可以监听焦点的改变事件，如下：

```dart
...
// 创建 focusNode   
FocusNode focusNode = new FocusNode();
...
// focusNode绑定输入框   
TextField(focusNode: focusNode);
...
// 监听焦点变化    
focusNode.addListener((){
  //获得焦点时为true，否则为false
   print(focusNode.hasFocus);
});
```

### 自定义样式

```dart
 decoration: InputDecoration(
      labelText: "输入名字",
      hintText: "我是一个提示",
      //未获得焦点时,设置下划线为灰色
      enabledBorder: UnderlineInputBorder(
          borderSide: BorderSide(color: Colors.grey, width: 5)),
      //获得焦点时,设置下划线为红色
      focusedBorder: UnderlineInputBorder(
          borderSide: BorderSide(color: Colors.red, width: 5)),
      prefixIcon: Icon(Icons.verified_user)),
)
```

上述的方式是通过 设置inputDecoration 的有无焦点状态下的下划线颜色，另外，我们还可以通过 **主题** 来定义输入框的样式。

TextField 默认的颜色使用的是我们默认的主题色，所以自然的，我们可以通过更改主题色来定义相应的全局颜色，或者说通过更改主题色来更改 TextField 的默认颜色。

如下示例：

![image-20201217235144214](https://tva1.sinaimg.cn/large/0081Kckwly1glragise8ej30nz0eadi5.jpg)



需要注意的是，我们通过主题定义的统一的下划线颜色和文字样式，当输入框在获取焦点时， **labelText** 不会高亮显示，而且还无法定义下划线宽度，另一种方式是直接隐藏掉 TextField 本身下划线，然后通过 Container 去嵌套。如下所示。

```dart
Container(
  child: TextField(
    decoration: InputDecoration(
        labelText: "测试3",
        hintText: "我是要隐藏下划线的TextField",
        border: InputBorder.none),
  ),
  decoration: BoxDecoration(
      border:
          Border(bottom: BorderSide(color: Colors.grey, width: 1))),
),
```



## 表单

在实际开发中，我们都会面对这对输入框数据进行合法性的校验，但是如果对每一个 **TextField** 都分别进行校验将非常麻烦。而且如果用户想清除一组 **TextField** 内容，一个一个清除就会非常痛苦，为此，Flutter 提供了 **From** 组件，它可以对输入框进行分组，然后进行一些统一操作，如 输入内容校验，输入框重置以及输入内容保存。

·