# Android属性动画无限旋转

两种方式实现：

### 方式1：

```java
ImageView img = findViewById(R.id.img_src);
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(img, "rotation", 0, 359);
objectAnimator.setRepeatCount(ValueAnimator.INFINITE);
objectAnimator.setDuration(2000);
objectAnimator.setInterpolator(new LinearInterpolator());
objectAnimator.start();
```
### 方式2：

```java
    RotateAnimation animation = new RotateAnimation(0f, 360f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
        animation.setFillAfter(true);
        animation.setRepeatCount(Animation.INFINITE);
        animation.setDuration(1000);
        animation.setInterpolator(new LinearInterpolator());
        img.setAnimation(animation);
        animation.start();
```
<img src= "https://tva1.sinaimg.cn/large/006tNbRwly1g9yenpv830g30u01o07wh.gif" height="600" width="300">





#### 解析：

```
RotateAnimation extend Animation   
```

用于控制对象旋转的动画。



```
ObjectAnimator extends ValueAnimator  ValueAnimator extends Animator
```

用于为目标动画提供属性支持。



**说简单点其实就是 Animation 和 Animator的区别：**

**Animation** 

在每次进行绘图的时候,通过对整块画布的矩阵进行变换,从而实现一种视图坐标的移动,但实际上其在 View 内部真实的坐标位置及其他相关属性始终恒定。



**Animator**

内部其实是通过 计算时间线特定该有的值，然后通过set get的方式实现内部属于更改，再通过 类似 invalidate 的方式刷新布局，从而实现动画效果。



#### **区别：**

- 最大区别莫过于，使用 **Animation** 之后，你原来的view位置其实并没有改变。而 **Animator** 因为改变了内部属性，所以位置实时改变。
- **Animator** 相对来说也更加强大，只要view自定义或者自带了set，get方法，那么就可以实现动画效果，说简单点  ***Animator 并不负责动画，它只是负责计算不同时间线该有的值，从而让用户自己去设置，可扩展性更强。***