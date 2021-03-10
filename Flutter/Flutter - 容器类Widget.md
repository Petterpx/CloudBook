# Flutter- 容器类Widget

容器类Widget与布局类Widget都一般作用于其子Widget,不同的是：

- 布局类Widget一般都需要接收一个 Widget数组(children),他们直接或间接继承自 MultiChildRenderObjectWidget ，比如我们常见的 Column->Flex->MultiChildRenderObjectWidget。而容器类Widget一般只需要接收一个子Widget(child)，他们直接或间接继承自(或包含)SingleChildRenderObjectWidget，比如 **DecoratedBox**
- 布局类 Widget 是按照一定的排列方式来对其子Widget进行排列，而容器类Widget一般只是包含其子Widget，对其添加一些修饰，变换，或限制大小。



## ConstrainedBox

尺寸限制类容器用于限制容器的大小。

### ConstrainedBox

用于对子组件添加额外的约束，比如，如果想让子组件的最小宽度是80像素，那么当增加了这个限制后，子组件无论设置多小，其最小宽度都会是80像素。

其有如下的参数。

```
 constraints: BoxConstraints(
          minWidth: 50, //最小宽度
          maxWidth: double.infinity, //最大宽度
          minHeight: 50,
          maxHeight: double.infinity),
```

其页提供了相应的快捷设置方法,便于快速设置。

![image-20210109115102903](https://tva1.sinaimg.cn/large/008eGmZEly1gmhavsfyc3j31ei0c4dky.jpg)



### DecoratedBox

DecoratedBox可以在其子组件绘制前后，或后绘制一些装饰，如背景，边框，渐变等。

```
class DecoratedBoxWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) => DecoratedBox(
        //DecoratedBox的一个子类，
        decoration: BoxDecoration(
            //渐变色
            gradient: LinearGradient(colors: [Colors.red, Colors.orange]),
            //圆角
            borderRadius: BorderRadius.circular(3.0),
            //背景混合模式
            // backgroundBlendMode:BlendMode.dst ,
            //背景色，如果存在gradient,则忽略这个属性
            color: Colors.blueGrey,
            //阴影
            boxShadow: [
              BoxShadow(
                  color: Colors.blue,
                  offset: Offset(2.0, 2.0),
                  blurRadius: 4.0)
            ],
            //形状，当circle时，和borderRadius不能共存
            shape: BoxShape.rectangle),
        //决定在哪里绘制Decoration,background 背景装饰，foreground前景装饰，即绘制在子组件之上
        position: DecorationPosition.background,
        child: Padding(
          padding: EdgeInsets.symmetric(horizontal: 80, vertical: 18),
          child: Text(
            "test",
            style: TextStyle(color: Colors.white),
          ),
        ),
      );
}

```

![image-20210109230431190](https://tva1.sinaimg.cn/large/008eGmZEly1gmhuciy0q6j30b005aaap.jpg)



### Transform(变换)

用于在子组件绘制时对其应用一些矩阵变换来实现一些特效， 其中 **Matrix4** 是一个4d矩阵库，通过其我们可以实现各种操作。

如下所示：

|                             代码                             |                             图示                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20210120204839283](../../../../Library/Application Support/typora-user-images/image-20210120204839283.png) | ![image-20210120204901328](https://tva1.sinaimg.cn/large/008eGmZEgy1gmug8w7ltxj30ia0bgt9n.jpg) |
| ![image-20210120212934265](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuhf5rmp2j30ws0hitdt.jpg) | ![image-20210120211956158](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuh522jbtj30ga08oaaf.jpg) |
| ![image-20210120212024620](../../../../Library/Application Support/typora-user-images/image-20210120212024620.png) | ![image-20210120212035114](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuh5qi89kj30es07iq38.jpg) |
| ![image-20210120212155092](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuh75mu7aj30xg0pmq97.jpg) | ![image-20210120212118804](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuh6i7jzxj30eg09kjrq.jpg) |

> 由于矩阵的变化只会作用在绘制阶段，所以某些场景下，在UI需要变化时，可以直接通过矩阵变化来达到视觉上的UI改变，而无需去重写触发 build。

#### 问题：

**使用 Transform 对其子组件先进行平移在旋转与先旋转再平移，两者的最终效果会一样吗？**

![image-20210120235543880](https://tva1.sinaimg.cn/large/008eGmZEgy1gmuln8he44j316g0j240x.jpg)

由上图可发现,当旋转后，相对应的x,y方向已经改变，所以先平移再旋转与先旋转后平移，对于矩阵的改变而言，其是不同的效果



#### RotatedBox

RotatedBox 与 Transform.rotate 功能相似，但不同的支出在于， **RotatedBox** 的变换是在layout阶段，会影响子widget的位置和大小。如下图所示：





