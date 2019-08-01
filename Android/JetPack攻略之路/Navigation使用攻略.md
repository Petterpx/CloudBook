# Navigation使用攻略

[官网链接](https://developer.android.google.cn/guide/navigation/navigation-getting-started)



## 使用步骤：

右击res,创建 navigation 文件夹，右击创建你的导航布局

![navigation-graph_2x-callouts](http://ww2.sinaimg.cn/large/006tNc79ly1g59u5ni7qvj30w70u0dj3.jpg)

按照你的方式随意组合，代码自动补齐。



最后，将 NavHost(类似于导航容器) 添加到 活动中。如下：

> 传统单 Activity 写法中，我们需要在这里写一个 Fragment 或者别的什么布局,在 Navigation 中 HavHostFragment 就是Fragment 的容器，布局中的 navGraph属性建立了与 navigaion资源文件的关系。

```java
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/nav_host_fragment"
          //包含实现的类名 NavHost
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:defaultNavHost="true"
          //与 NacHostFragment 关联。即指定NavHostFragment用户可以导航到的所有目标
        app:navGraph="@navigation/nav_graph" />

</androidx.constraintlayout.widget.ConstraintLayout>
```



MainActivity

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    /**
     * 委托back事件
     * 若栈中有两个以上fragment,点击back会返回到上一个Fragment
     * @return
     */
    @Override
    public boolean onSupportNavigateUp() {
        Fragment fragment=getSupportFragmentManager().findFragmentById(R.id.nav_host_fragment);
        assert fragment != null;
        return NavHostFragment.findNavController(fragment).navigateUp();
    }
}

```



Fragment1

```java

public class BlankFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_blank, container, false);
    }


    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        view.findViewById(R.id.btn_1).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Navigation.findNavController(view).navigate(R.id.blankFragment2);
            }
        });
    }
}
```

Fragment2

```java
public class BlankFragment2 extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_blank_fragment2, container, false);
    }
    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        view.findViewById(R.id.btn_2).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Navigation.findNavController(view).navigateUp();
            }
        });
    }
}
```

