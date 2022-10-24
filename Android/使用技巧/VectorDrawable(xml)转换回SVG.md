# VectorDrawable(xml)转换回SVG

[原文链接](https://sarasarasa.net/post/396831a4.html) 

Android 用的矢量图是 VectorDrawable（xml格式）。

如果我们想对已经转换成 VectorDrawable 的矢量图进行修改的话，

最好先转换回 SVG 格式再使用 inkscape 之类的矢量图图形编辑工具进行修改。

# 步骤

1. **将头部的：**

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <vector xmlns:android="http://schemas.android.com/apk/res/android"
   ```

   **替换成**

   ```
   <svg xmlns="http://www.w3.org/2000/svg"
   ```

   闭标签也做相应修改。

2. **将`android:width`替换成`width`**

3. **将`android:height`替换成`height`**

4. **将`android:pathData`替换成`d`。**

5. **将`android:fillColor`替换成`fill`。**

   如果没有`android:fillcolor`的话，要加上`fill="#ffffff"`

6. **将`android:viewportHeight="24" android:viewportWidth="24"`替换成`viewBox="0 0 24 24"`的形式。**

# 例子

Vector Drawable

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportHeight="24"
    android:viewportWidth="24">

    <path
        android:fillColor="#ffffff"
        android:pathData="M12,3L2,12h3v8h2.5v-0.8c0-1.5,3-2.2,4.5-2.2s4.5,0.8,4.5,2.2V20H19v-8h3L12,3zM12,15.2c1.2,0-2.2-1-2.2-2.2 s1-2.2,2.2-2.2s2.2,1,2.2,2.2S13.2,15.2,12,15.2z" />
    <path android:pathData="M0,0h24v24H0V0z" />
</vector>
```

SVG

```
<svg xmlns="http://www.w3.org/2000/svg"
    width="24" 
    height="24" 
    viewBox="0 0 24 24" >

    <path 
        fill="#ffffff"
        d="M12,3L2,12h3v8h2.5v-0.8c0-1.5,3-2.2,4.5-2.2s4.5,0.8,4.5,2.2V20H19v8h3L12,3zM12,15.2c1.2,0-2.2-1-2.2-2.2 s1-2.2,2.2-2.2s2.2,1,2.2,2.2S13.2,15.2,12,15.2z" />
    <path d="M0,0h24v24H0V0z" fill="none"/>
</svg>
```