# TabLayout——你应该知道的一些方法。

### 与ViewPager的搭配使用。

布局：

```xml
<com.google.android.material.tabs.TabLayout
    android:id="@+id/tl_add_vp"
   	//导航器长度与title同步
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    //导航器颜色
    app:tabIndicatorColor="@android:color/white"
    //导航器高度                     
    app:tabIndicatorHeight="2dp"
    //允许tab滑动
    app:tabMode="scrollable"
    //设置tab按下时的背景颜色
    app:tabRippleColor="@android:color/transparent"
    //设置tab字体大小(需要注意的是需要在style中设置，方可使用)
    app:tabTextAppearance="@style/TabLayoutTextStyle"
    //tab字体颜色                                       
    app:tabTextColor="@android:color/white" />
```



java代码

```java
List<Fragment> list = new ArrayList<>();
list.add(new ConsumeFragment());
list.add(new ConsumeFragment());
String [] sums={"支出","收入"};
ConsumePagerAdapter adapter = new ConsumePagerAdapter(getChildFragmentManager(), list,sums);
viewPager.setAdapter(adapter);
//监听事件
viewPager.addOnPageChangeListener(this);
//设置tab标签
tabLayout.addTab(tabLayout.newTab().setText("支出"));
tabLayout.addTab(tabLayout.newTab().setText("收入"));
//与ViewPager关联
tabLayout.setupWithViewPager(viewPager);
```

viewpager适配器：（重写getPageTitle 方法）

```
public class ConsumePagerAdapter extends FragmentStatePagerAdapter {
    private List<Fragment> list;
    private String[] sums;

    public ConsumePagerAdapter(FragmentManager fm, List<Fragment> list, String[] sums) {
        super(fm);
        this.list = list;
        this.sums = sums;
    }


    @Override
    public Fragment getItem(int position) {
        return list.get(position);
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Nullable
    @Override
    public CharSequence getPageTitle(int position) {
        return sums[position];

    }
}
```

