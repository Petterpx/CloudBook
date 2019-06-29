# 关于EditText 焦点问题

```java
XML
android:descendantFocusability="blocksDescendants"

beforeDescendants：viewgroup会优先其子类控件而获取到焦点
afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点
blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点


代码处
visable.setDescendantFocusability(FOCUS_AFTER_DESCENDANTS);
```

我们有些时候的需求是，EditText在不需要的时候，无法点击，或者取消它的默认焦点。

### 取消EditText的焦点

 setFousable()  //设置该视图是否可以接收焦点
 setFocusableInTouchMode();  //设置该视图在触摸模式下是否可以接收焦点 

​		这里的详细解释借鉴别人的解释。
​		类似非触屏手机时代，需要使用键盘的上下左右去选中某个应用，然后点击确定执行。而触屏手机，我们只需要对应用点击一次，即可，无需焦点。也就是会所焦点是为了标记你目前选中的位置的。而这个在日历中却是有用的。
android:focusable与android:focusableInTouchMode
前者针对在键盘下操作的情况，如果设置为true，则键盘上下左右选中，焦点会随之移动。
而后者，显然是针对触屏情况下的，也就是我们点击屏幕的上的某个控件时，不要立即执行相应的点击逻辑，而是先显示焦点（即控件被选中），再点击才执行逻辑。
android:focusable=“true”不会改变android:focusableInTouchMode，因此只在键盘状态下显示焦点，在TouchMode状态下，依旧无法显示焦点。
android:focusable=“false”，一定会使android:focusableInTouchMode=“false”。
相对的
android：focusableInTouchMode=“false”,不会影响android：focusable。

android:focusableInTouchMode=”true”,一定会是android：focusable=“true”

直接上代码

```
代码：
editText.setFocusable(false)
editText.setFocusableInTouchMode(false);
	  
xml  
android:focusable="false"
android:focusableInTouchMode="false"
```



下面上一个Demo

比如我们有EditText，由一个switch控制，当switch关闭时，editText可以输入，有焦点，否则无法点击，无焦点。

按照我们上面取消EditText焦点的例子，如果我们这里有5个EditText，很多人会写出这样的例子

```
EditText e1;
...

private void setFoucus(Boolean foucus){
    e1.setFoucusable(fouces)
    e1.setFocusableInTouchMode(focus)
    e2.setFoucusable(fouces)
    e2.setFocusableInTouchMode(focus)
    ...
    e5.setFoucusable(fouces)
    e5.setFocusableInTouchMode(focus)
}
简化版
用List来保存对象，然后for遍历，但是你的EditText，需要声明多少个呢
```

当我们的EditText 1两个的时候，没有问题，可是当我们有十几个输入框的时候呢？

现在我们用另一个办法。将这些输入框放在同一个线性布局里，然后利用 setDescendantFocusability() 方法，设置子类控件与viewgroup之间的焦点关系。

Demo如下

```
 private LinearLayout layout;		//父布局实例
 ...
 private Boolean fouces=false;		//默认switch状态为false
 aSwitch.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (fouces) {
                    layout.setDescendantFocusability(FOCUS_AFTER_DESCENDANTS);
                    fouces=false;
                } else {
                    layout.clearFocus();	//清除fouces
                    layout.setDescendantFocusability(FOCUS_BLOCK_DESCENDANTS);
                    fouces=true;
                }
            }
        });
        
        arent(ViewGroup)中存储的mFocus设置为null,即清除焦点
    2. rootViewRequestFocus();// 调用DecorView的requestFocus()方法,作用是找到视图中的一个View,
```

看一下效果

![Honeycam 2019-02-22 21-41-08](C:\Users\15094\Documents\Honeycam\Honeycam 2019-02-22 21-41-08.gif)

但是这样有个问题，clearFocus（）方法好像失效了，没什么用。



先不急，我们先看一下clearFocus的实现

```
  @Override
    public void clearFocus() {
        if (DBG) {
            System.out.println(this + " clearFocus()");
        }
        if (mFocused == null) {
            super.clearFocus();
        } else {
            View focused = mFocused;
            mFocused = null;
            focused.clearFocus();
        }
    }
    不管这些，我们顺着最后的调用方法走 focused.clearFocus();
    
     public void clearFocus() {
        if (DBG) {
            System.out.println(this + " clearFocus()");
        }

        final boolean refocus = sAlwaysAssignFocus || !isInTouchMode();
        clearFocusInternal(null, true, refocus);
    }
    
   这里的意思是，如果焦点可用，或者非触控模式下，焦点会尝试将焦点放在第一个可以对焦的视图上，也就是说，相当于它被重置了，所以产生了我们上面图片里的问题，焦点没有被清除。
     接着继续顺着代码走 clearFocusInternal(null, true, refocus);
     
     void clearFocusInternal(View focused, boolean propagate, boolean refocus) {
        if ((mPrivateFlags & PFLAG_FOCUSED) != 0) {
            mPrivateFlags &= ~PFLAG_FOCUSED;
            clearParentsWantFocus();

            if (propagate && mParent != null) {
                mParent.clearChildFocus(this);
            }

            onFocusChanged(false, 0, null);
            refreshDrawableState();

            if (propagate && (!refocus || !rootViewRequestFocus())) {
                notifyGlobalFocusCleared(this);
            }
        }
    }
    这里清除视图中的焦点，如果propagate为true，可选地将更改向上传播到父层次结构，并放置新的焦点。
    
    总结一下，也就是我们需要在父布局处添加 触控模式为true，即就是android:focusableInTouchMode="true"，这样当清除焦点的时候，就会将焦点赋给父布局，而不是重置到第一个EditText.到了现在，我们可以尝试一下，如果设置第一个输入框focusableInTouchMode为false,那么当你点击了别的输入框，然后点击switch,会发现，焦点会在第二个输入框，而不会在第一个。
```

找到问题所在