# BottomNavigationView使用笔记

Android底部导航栏组件，通常适用于3个到5个tab.

使用方法：

xml:

```xml
<android.support.design.widget.BottomNavigationView
        android:id="@+id/bottomnavigationview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:itemIconTint="@drawable/selector_tab_color"
        app:itemTextColor="@drawable/selector_tab_color"
        app:menu="@menu/bottom_navigation_tab">

    </android.support.design.widget.BottomNavigationView>
```

**属性如下：**

- **itemIconTint：icon图片的颜色**
- **itemTextColor：文本的颜色**
- **app:iteamBackground 指的是底部导航栏的背景颜色,默认是主题的颜色**

**menu:tab的布局：**selector_tab_color：

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_checked="true" android:color="@color/colorAccent"/>
    <item android:state_checked="false" android:color="@color/colorPrimary"/>
</selector>
```

一般使用的时候，图片的颜色和文字的颜色是一致的。



**接下来新建菜单：**

res下新建menu文件夹,新建你的底部导航栏xml文件：menu_bootom.xml:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_message"
        android:enabled="true"
        android:icon="@drawable/liebiao"
        android:title="列表"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/menu_contacts"
        android:enabled="true"
        android:icon="@drawable/dingdan"
        android:title="订单"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/menu_discover"
        android:enabled="true"
        android:icon="@drawable/zhifu"
        android:title="支付"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/menu_me"
        android:enabled="true"
        android:icon="@drawable/guanyu"
        android:title="关于"
        app:showAsAction="ifRoom" />
</menu>
```

代码里使用：

```java
BottomNavigationView bottomNavigationView = findViewById(R.id.bottom_view);
bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {

        switch (menuItem.getItemId()) {
            case R.id.menu_message:
               
                break;
            case R.id.menu_contacts:
                
                break;
            case R.id.menu_discover:
               
                break;
            case R.id.menu_me:
               
                break;
            default:
                break;
        }
			//这里一定要返回true.否则点击之后没有效果。
        return true;
    }
});
```



需要注意的是，在使用时，底部tab个数如果>=3,文字图标可能不会显示，这个时候，需要给xml中添加以下属性：

```xml
app:labelVisibilityMode="labeled"
```

更多使用技巧参阅：[BottomNavigationView更好使用](http://androidcookie.com/bottomnavigationview%E5%A7%8B%E7%BB%88%E6%98%BE%E7%A4%BA%E5%9B%BE%E6%A0%87%E5%92%8C%E6%96%87%E6%9C%AC%E6%A0%87%E7%AD%BE.html)



