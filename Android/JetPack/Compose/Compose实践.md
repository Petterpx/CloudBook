# Compose实践



## Compose中的常用组件

### Column

垂直列表,类似于Android中的LinearLayout.具体示例如下：

属性如下：

```kotlin
Column(
  	//修饰符
    modifier: Modifier = Modifier,
  	//垂直排列方向
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
  	//水平排列方向
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
  	//具体的内容
    content: @Composable ColumnScope.() -> Unit
)
```

![image-20210311214807976](https://tva1.sinaimg.cn/large/008eGmZEgy1gogaxtiii3j322c0u012z.jpg)



对于 `verticalArrangement` 与  `horizontalAlignment` ，除了提供的通用的 **Start,End,Top** 等等 ，你还可以通过如下方式自定义具体的内部view摆放位置。

#### Arrangement

> **用于指定布局子项在主轴方向的排列。**

Arrangement.aligned(BiasAlignment.Vertical(xx))

其中 [0] 表示居中，[1] 表示在底部 ,[-1] 表示在顶部 ,[-1,1]范围之内则表示自定义的区间

```
//垂直方向靠左显示
Arrangement.aligned(BiasAlignment.Vertical(-1f))
//垂直方向居中显示
Arrangement.aligned(BiasAlignment.Vertical(0F))
//垂直方向靠右显示
Arrangement.aligned(BiasAlignment.Vertical(1f))
```





#### BiasAlignment

> 由偏移量指定的[Alignment]：例如，偏移量-1表示与起始位置的对齐，偏移量0表示居中，偏移量1表示末尾。可以指定任何值以获得对齐方式。在[-1，1]范围内，获得的对齐方式会将对齐的尺寸完全定位在可用空间内，而在范围之外，它将对齐的尺寸部分或完全定位在外部。



### Divider

分割线





