# Componse教程 | 关于 Alignment与Arrangement



## 引言

关于Alignment和Arrangement，熟悉的人都知道，这两个接口一般用于排列我们View的方向，但是很多刚开始学习Componse的同学容易陷入不知道该怎么用，而官方的资料现阶段又很少。故写此篇，以便帮助大家扫清入门道路上的问题。



## Alignment

通常用于定义父布局内子布局的对其方式。

内部包含了如下两个接口：

- Horizontal

  用于定义水平方向对齐方式

- Vertical

  用于定义竖直方向对齐方式

与 Alignment 相关的有两个具体实现类。

- AbsoluteAlignment (偏差对齐)
- BiasAbsoluteAlignment (绝对偏差对齐)

怎么理解呢，别着急，跟着我下面的Demo,一看便知。

### AbsoluteAlignment

