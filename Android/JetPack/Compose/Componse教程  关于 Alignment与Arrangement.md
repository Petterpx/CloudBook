# Componse教程 | 关于 Alignment与Arrangement



## 引言

关于Alignment和Arrangement，这两个接口一般用于排列我们Componse中组件的方向，但是很多刚开始学习Componse的同学容易陷入不知道该怎么用，而官方的资料现阶段又很少。故写此篇，以便一个快速理解。

> 原以为会写的很复杂，最后发现，这个并没有想象的那么繁琐，于是就简单写了一下我的理解。由于目前水平有限，后续如果有发现错误的地方，也会及时修正。
>



## Alignment

通常用于一个布局在父布局中的对齐方式，可以认为就是用来做水平方向的对齐方式；

内部包含了如下两个接口：

- Horizontal

  用于定义横轴方向对齐方式

- Vertical

  用于定义竖直方向对齐方式

与 Alignment 相关的有两个具体实现类。

- AbsoluteAlignment (偏差对齐)
- BiasAbsoluteAlignment (绝对偏差对齐)

### AbsoluteAlignment

```kotlin
// 2D AbsoluteAlignments. 即用于方向不确定的时候，内部具体的排列方式
val TopLeft: Alignment = BiasAbsoluteAlignment(-1f, -1f)
val TopRight: Alignment = BiasAbsoluteAlignment(1f, -1f)
val CenterLeft: Alignment = BiasAbsoluteAlignment(-1f, 0f)
val CenterRight: Alignment = BiasAbsoluteAlignment(1f, 0f)
val BottomLeft: Alignment = BiasAbsoluteAlignment(-1f, 1f)
val BottomRight: Alignment = BiasAbsoluteAlignment(1f, 1f)
// 1D BiasAbsoluteAlignment.Horizontals. 用于布局方向确定时，水平与竖直的排列方式
val Left: Alignment.Horizontal = BiasAbsoluteAlignment.Horizontal(-1f)
val Right: Alignment.Horizontal = BiasAbsoluteAlignment.Horizontal(1f)
```

google提供好的对齐方式静态类，便于不知道布局方向时的使用。

### BiasAlignment

```kotlin
data class BiasAlignment(
    val horizontalBias: Float,
    val verticalBias: Float
){
  - class Horizontal(bias) 水平方向对齐方式
  - class Vertical(bias)  竖直方向对齐方式
}
```

按照偏移量而定的对齐方式,-1 表示start或者top,0表示居中，1表示end或者bottom



### BiasAbsoluteAlignment

```kotlin
data class BiasAbsoluteAlignment(
    private val horizontalBias: Float,
    private val verticalBias: Float
){
   - class Horizontal(bias) 水平对齐方式
  	bias- [-1,1] -1表示左对齐，0表示居中，1表示右对齐
}
```

与BiasAlignment 不同的是，-1表示对齐到左上角，0表示居中，1表示右下角





## Arrangement

用于指定父布局的子元素在布局内主轴方向的对齐方式：



### 相同点

上面的 ment 除了**AbsoluteAlignment**  , 都均包含了两个子类，**Horizontal** 和 **Vertical**

#### 为什么呢？

因为对于一个View来说,布局方向存在水平或者竖直两种，所以存在上述两个子类，以便于调用。



## 示例

[本文示例代码地址]([示例代码地址](https://github.com/Petterpx/ComposeSimple/blob/master/app/src/main/java/com/compose/simple/ui/layout/ColumnSimple.kt))

```kotlin
@Composable
fun ColumnSimple(
    text: String,
    modifier: Modifier = Modifier,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start
) {
    Column(
        modifier = modifier
            .background(color = Color.Yellow)
            .padding(5.dp),
        verticalArrangement, horizontalAlignment
    ) {
        Text(text = text, color = Color.Black)
    }
}
```



### 默认的排列方向

| 示例                                                         | 效果图                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20210318222119082](https://tva1.sinaimg.cn/large/008eGmZEly1goof8i2sqxj30mt05s0tc.jpg) | ![image-20210318222126651](https://tva1.sinaimg.cn/large/008eGmZEly1goof8mnepnj30de0410sp.jpg) |

### 自定义竖直的排列方式

|                             示例                             |                             效果                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20210318222307519](https://tva1.sinaimg.cn/large/008eGmZEly1goofadetqgj30x10r6td6.jpg) | ![image-20210318222315267](https://tva1.sinaimg.cn/large/008eGmZEly1got0ppdjdkj30d7084mx8.jpg) |

### 自定义水平排列

| 示例                                                         | 效果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20210318222453101](https://tva1.sinaimg.cn/large/008eGmZEly1goofc7bdvwj30qa0qcdju.jpg) | ![image-20210318222525108](https://tva1.sinaimg.cn/large/008eGmZEly1goofcrfk2mj30d30dn3ym.jpg) |



## 

