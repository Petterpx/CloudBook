# 解决Android WebVIew中视频播放前出现的灰色三角形播放图标

最近在开发中，发现WebView中播放视频时，会出现灰色的播放图标，如图：

![image-20200317224339103](https://tva1.sinaimg.cn/large/00831rSTly1gcxb2y10hbj30qa0f4dhy.jpg)



解决办法如下：

重写 ***WebChromeClient*** 类中的	***getDefaultVideoPoster*** 方法，返回一个透明的bitmap.

![image-20200317225241398](https://tva1.sinaimg.cn/large/00831rSTly1gcxbcc432kj312s0ge76z.jpg)



***getDefaultVideoPoster***

```
不播放时，视频元素由“海报”图像表示。可以通过* HTML中视频标签的poster属性指定要使用的*图片。如果该属性不存在，则将使用默认海报。此*方法允许ChromeClient提供该默认图像。 * * @返回位图用作默认海报的图像；如果没有可用的图像，则为{@code null}。
```

