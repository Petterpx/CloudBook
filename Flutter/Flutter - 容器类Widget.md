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



