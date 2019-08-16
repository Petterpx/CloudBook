Android布局优化-viewSwitcher的使用

他的作用用于处理不同view的隐藏与显示，如果你有两个view,同时只能显示一个，比较通俗的做法是 使用Gone或者VISIBLE, viewSwitcher的出现就是用来优雅的处理这个。

上代码：

一个简单的布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".Main2Activity">

    <Button
        android:id="@+id/btn_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="1" />

    <Button
        android:id="@+id/btn_2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="2" />

    <ViewSwitcher
        android:id="@+id/view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btn_view_1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="显示1" />

        <TextView
            android:id="@+id/tv_view_1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="123" />
    </ViewSwitcher>
</LinearLayout>
```

```java
package com.example.navationdemo;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.view.animation.Animation;
import android.view.animation.AnimationUtils;
import android.widget.Button;
import android.widget.TextView;
import android.widget.ViewSwitcher;

public class Main2Activity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        final ViewSwitcher viewSwitcher = findViewById(R.id.view);
        //加动画
        Animation slide_in_left = AnimationUtils.loadAnimation(this, android.R.anim.slide_in_left);
        Animation slide_out_right = AnimationUtils.loadAnimation(this, android.R.anim.slide_out_right);
        viewSwitcher.setInAnimation(slide_in_left);
        viewSwitcher.setOutAnimation(slide_out_right);
        viewSwitcher.setVisibility(View.VISIBLE);

        findViewById(R.id.btn_1).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                viewSwitcher.setDisplayedChild(0);
            }
        });
        findViewById(R.id.btn_2).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                viewSwitcher.setDisplayedChild(1);
            }
        });


    }
}
```

代码如上，使用很简单。