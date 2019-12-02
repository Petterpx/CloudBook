# LottieFiles使用记录

简单解释，一种简单的动画处理方式，时候json文件显示动画，避免多余的操作。需要UI同学导出相应的动画 Json 文件，一些复杂的 UI的动画 可以 通过其完成。

[官方文档](http://airbnb.io/lottie/#/android)

支持从以下位置加载动画

- 中的json动画`src/main/res/raw`。
- 中的json文件`src/main/assets`。
- 中的zip文件`src/main/assets`。查看[图片文档](http://airbnb.io/lottie/#/android?id=images)以获取更多信息。
- json或zip文件的网址。
- json字符串。来源可以来自任何东西，包括您自己的网络堆栈。
- 一个JSON文件或zip文件的InputStream。





#### 导入：

```java
implementation 'com.airbnb.android:lottie:3.2.2
```



#### 开发使用：

![image-20191202141934558](https://tva1.sinaimg.cn/large/006tNbRwly1g9ictsgucij30kg0kqwgl.jpg)

xml

```xml
 <com.airbnb.lottie.LottieAnimationView
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:id="@+id/lottie_layer_name"
        app:lottie_autoPlay="true"
        app:lottie_fileName="123.json"
        app:lottie_loop="true"
        tools:ignore="MissingConstraints" />
```



代码中

```
     LottieAnimationView animationView=findViewById(R.id.lottie_layer_name);
     		//设置图片文件夹
//        animationView.setImageAssetsFolder("");
        //设置Json文件，如果xml没指定的话
        animationView.setAnimation("123.json");
        //无线循环
        animationView.loop(true);
        //开启动画
        animationView.playAnimation();
        
        //当前是否在播放
       	animationView.isAnimating()
```

```java
        //监听状态
//        lottieAnimationView.addAnimatorListener(new Animator.AnimatorListener() {
//            @Override
//            public void onAnimationStart(Animator animation) {
//                Log.e("demo","启动");
//            }
//
//            @Override
//            public void onAnimationEnd(Animator animation) {
//                Log.e("demo","结束");
//            }
//
//            @Override
//            public void onAnimationCancel(Animator animation) {
//                Log.e("demo","关闭");
//            }
//
//            @Override
//            public void onAnimationRepeat(Animator animation) {
//                Log.e("demo","重启");
//            }
//        });
```



效果：

![Jietu20191202-141810-HD](/Users/petterp/Downloads/Jietu20191202-141810-HD.gif)





#### 通过Uri加载动画

```java
animationView.setAnimationFromUrl("https://assets1.lottiefiles.com/packages/lf20_gYEXhD.json");
animationView.playAnimation();
```



#### 通过 Json字符串加载

```
 String json="json";
 animationView.setAnimationFromJson(json,"huan");
 animationView.playAnimation();
```

