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

