# DrawLayout实现滑动时所在layout滑动效果。

监听DrawLayout滑动状态，实时改变位置即可。

```java
drawerLayout.addDrawerListener(new DrawerLayout.DrawerListener() {
    @Override
    public void onDrawerSlide(@NonNull View drawerView, float slideOffset) {
        //获取屏幕的宽高
        WindowManager manager = (WindowManager) getProxyActivity().getSystemService(Context.WINDOW_SERVICE);
        Display display = manager.getDefaultDisplay();
        //设置右面的布局位置  根据左面菜单的right作为右面布局的left   左面的right+屏幕的宽度（或者right的宽度这里是相等的）为右面布局的right
        right.layout(left.getRight(), 0, left.getRight() + display.getWidth(), display.getHeight());
    }

    @Override
    public void onDrawerOpened(@NonNull View drawerView) {

    }

    @Override
    public void onDrawerClosed(@NonNull View drawerView) {

    }

    @Override
    public void onDrawerStateChanged(int newState) {

    }
});
```

看看效果：

![Jietu20190725-202054-HD](http://ww4.sinaimg.cn/large/006tNc79ly1g5ccqf9tv0g307u0gn7wk.gif)

