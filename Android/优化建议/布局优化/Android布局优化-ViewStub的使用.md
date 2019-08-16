Android布局优化-ViewStub的使用

解释转载自https://www.cnblogs.com/lwbqqyumidi/p/4047108.html

ViewStub是Android布局优化中一个很不错的标签/控件，直接继承自View。

ViewStub可以理解成一个非常轻量级的View，与其他的控件一样，有着自己的属性及特定的方法。当ViewStub使用在布局文件中时，当程序inflate布局文件时，ViewStub本身也会被解析，且占据内存控件，但是与其他控件相比，主要区别体现在以下几点：

1.当布局文件inflate时，ViewStub控件虽然也占据内存，但是相相比于其他控件，ViewStub所占内存很小；

2.布局文件inflate时，ViewStub主要是作为一个“占位符”的性质，放置于view tree中，且ViewStub本身是不可见的。ViewStub中有一个layout属性，指向ViewStub本身可能被替换掉的布局文件，在一定时机时，通过viewStub.inflate()完成此过程；

3.ViewStub本身是不可见的，对ViewStub setVisibility(..)与其他控件不一样，ViewStub的setVisibility 成View.VISIBLE或INVISIBLE如果是首次使用，都会自动inflate其指向的布局文件，并替换ViewStub本身，再次使用则是相当于对其指向的布局文件设置可见性。

这里需要注意的是：

1.ViewStub之所以常称之为“延迟化加载”，是因为在教多数情况下，程序无需显示ViewStub所指向的布局文件，只有在特定的某些较少条件下，此时ViewStub所指向的布局文件才需要被inflate，且此布局文件直接将当前ViewStub替换掉，具体是通过viewStub.infalte()或viewStub.setVisibility(View.VISIBLE)来完成；

2.正确把握住ViewStub的应用场景非常重要，正如如1中所描述需求场景下，使用ViewStub可以优化布局；

3.对ViewStub的inflate操作只能进行一次，因为inflate的时候是将其指向的布局文件解析inflate并替换掉当前ViewStub本身（由此体现出了ViewStub“占位符”性质），一旦替换后，此时原来的布局文件中就没有ViewStub控件了，因此，如果多次对ViewStub进行infalte，会出现错误信息：ViewStub must have a non-null ViewGroup viewParent。



**我们只需要明白，ViewStub真正用于哪些不是特别容易显示的布局，比如首页启动后的的title设置。**

示例Demo:

xml

```java
<ViewStub
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/scr_stub_analysis"
    android:background="@android:color/white"
    android:layout="@layout/stub_analysis_data"
    />
<ScrollView
    android:id="@+id/scr_analysis_data"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
```



代码中：

```java
viewStub=findView...
View stubView;
if (stubView == null) {
    stubView = viewStub.inflate();
}
```

这里要注意，infalte方法只可调用一次，不可重复调用。否则报错

原因：因为inflate的时候是将其指向的布局文件解析inflate并替换掉当前ViewStub本身（由此体现出了ViewStub“占位符”性质），一旦替换后，此时原来的布局文件中就没有ViewStub控件了，因此，如果多次对ViewStub进行infalte，会出现错误信息：ViewStub must have a non-null ViewGroup viewParent。

