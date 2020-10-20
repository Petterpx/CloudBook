# DataBinding相关注解

## BindAdapter(扩展作用)

> 用来给指定View增加额外的属性。
>
> 比如当你没有 updateName 这个字段的时候。就可以新增它，这样在布局中就可以随意使用。

```kotlin
@BindingAdapter("updateName")
fun TextView.updateName(name: String) {
    text = name
}

//推荐使用下面这中带前缀的使用方式
@BindingAdapter("app:logTools")
fun TextView.logTools(name:String){
   Log.e("demo",name)
}

//默认 requireAll =true，必须全部参数都满足方可使用，如下xml
@SuppressLint("SetTextI18n")
@BindingAdapter("parameter1", "parameter2", requireAll = true)
fun TextView.testDoubleParameter(name: String, sum: Int) {
    text = "$name------$sum"
}
```

使用中

```XML
  <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="50dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@+id/btnStart"
    app:parameter1="@{viewModel.sumLiveData.toString()}"
    app:parameter2="@{viewModel.sumLiveData}"
    //因为这里没做限制，所以可以用任何前缀，前提在你的layout中增加
    wangdachui:updateName="@{viewModel.sumLiveData.toString()}" />

  示例：
  <layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
       //新增的在这里
    xmlns:wangdachui="http://schemas.android.com/tools"
       //
    xmlns:tools="http://schemas.android.com/tools">
```



## BindingMethods(替换作用)

> 用于当View中的某个属性与其对应的 set方法名称不对应时进行映射，如TextView 的属性 android:textColorHint 与之作用相同的方法是 setHintTextColor,此时属性名称与对应的 set 方法不一致，就需要使用 BindingMethods 注解将该属性与对应的 set 方法绑定，这样databinding 就能够按照属性值找到对应的 set 方法。

> 需要注意的是BindingMethods 并不能单独使用，需要配合BindingMethod才可以使用，如下Databinding库的处理：

![image-20201020162103811](https://tva1.sinaimg.cn/large/007S8ZIlly1gjvvhrpe6sj311d0evwj0.jpg)





## BindingConversion（类型转换）

> 用于类型的自动转换,在kotlin中使用时记得增加注解

```kotlin
object TextBindConversion {
    @JvmStatic
    @BindingConversion
    fun colorToDrawable(isType: String): Drawable {
        return when (isType) {
            "黄色" -> ColorDrawable(Color.RED)
            else -> ColorDrawable(Color.BLACK)
        }
    }
}
```

```xml
 <ImageView
     android:layout_width="100dp"
     android:layout_height="100dp"
     android:layout_marginTop="50dp"
     android:background='@{"黄色"}'
     app:layout_constraintEnd_toEndOf="parent"
     app:layout_constraintStart_toStartOf="parent"
     app:layout_constraintTop_toBottomOf="@+id/btnStart" />
```





