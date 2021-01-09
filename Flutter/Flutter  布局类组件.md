# Flutter | 布局类组件

## 基础概念

布局类组件都会包含一个或多个子组件，不同的布局类组件对子组件排版 (layout) 不同，在Flutter 中，根据 Widget 是否需要包含子节点将 Widget 分类3类，分别对应如下三种 Element,如下所示：

| Widget                        | 对应的Element                  | 作用                                                         |
| ----------------------------- | ------------------------------ | ------------------------------------------------------------ |
| LearRenderObjectWidget        | LeafRenderObjectElement        | Widget树的叶子节点，用于没有子节点的Widget,通常基础组件都属于这个，如Image |
| SingleChildRenderObjectWidget | SingleChildRenderObjectElement | 包含一个Widget,如 ConstrainedBox,DecoratedBox等              |
| MuitiChildRenderObjectWidget  | MultiChildRenderObjectElement  | 包含多个子Widget,一般都有一个children参数，接收一个Widget数组，如Row,Column,Stack等 |

> Flutter中很多Widget是直接继承自 StatelessWidget 或 StatefulWidget ，然后在 build() 方法中构建真正的 RenderObjectWidget,如Text,它其实是继承自 StatelessWidget,然后在 build() 方法中通过 RichText 来构建其子树，而RichText才是继承自 MultiChildRenderObjectWidget， 所以我们甚至可以认为 Text 就是隶属于MultiChildxxx。
>
> 所以所谓的 StatelessWidget 和 StatefulWidget 就是两个用于组合Widget的基类，它们本身并不关联最终的渲染对象(RenderObjectWidget).

而相应的布局类组件就是直接或间接继承(包含)**MuitChildxxx** 的Widget,它们一般都会有一个 children 属性用于接收子 Widget.



## Flow

Flow 主要用于一些需要自定义布局策略或性能要求比较高(如动画)的场景，Flow有如下优点：

- 性能好；Flow 是一个对子组件尺寸以及位置调整非常高效的控件，Flow 用转换矩阵在对子组件进行位置调整的时候进行了优化；在Flow 定位过后，如果子组件的尺寸或者位置发生了变化，在FlowDelegate 中的 paintChildren() 方法中调用 context.paintChild() 进行重绘，而 context.paintChild 在重绘时使用了转换矩阵，并没有实际调整组件位置。
- 灵活：由于我们需要自己实现 FlowDelegate 的painChilren() 方法，所以需要手动去计算每一个组件的位置，因此，具体的布局逻辑可以由我们定义。

缺点：

- 使用复杂，比较不易使用；
- 不能自适应组件大小，必须通过制定父容器大小或实现 FlowDelegate的 getSize 返回固定大小。





