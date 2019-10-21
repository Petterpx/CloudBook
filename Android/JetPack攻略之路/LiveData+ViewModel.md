# LiveData+ViewModel

> **LiveData与ViewModel 是JectPack组件中的一部分，使用起来也非常简单。**
>
> **LiveData 可以帮助你实现数据的更新监测，并配合 Activity 或者  Fragment的生命周期方法，避免出现view不在活动状态时更新数据。**
>
> **ViewModel 是一个数据持有类，因为生命周期的关系，它可以做到在你 Activity 异常退出时，自动保存你相应的数据，在Activity重启时再次展现，帮你解决数据自动恢复等问题。**



## 使用教程：

**导入依赖**

```xml
def lifecycle_version = "2.1.0"
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
```



**首先编写你的ViewModel类**

```java
/**
 * Created by Petterp
 * on 2019-10-19
 * Function: Name ViewModel
 *
 * 注意： 请将 LiveData 更新的UI的 ViewModel 对象存储在对象中，而不是活动或者片段：
 * 避免 activities 臃肿。view只负责显示数据
 *
 * onCreate方法中观察 LitveData 对象，并确保不会在 onResume方法中进行多余调用
 * 当 生命周期处于 onStart时，应用会从 LiveData 正在观察的对象中接收最新值。(注意：仅当 LiveData 已设置要观察的对象时，才会发生这种情况)
 * 更新的情况有以下 ： 从 非活动-> 活动状态，仅有一次；  数据更改时 -> 通知数据更新
 */
public class NameViewModel extends ViewModel {

    //类型为String的ViewModel
    private MutableLiveData<String> currentName;

    /**
     * 返回LiveData对象
     * @return
     */
    public MutableLiveData<String> getCurrentName() {
        if (currentName == null) {
            currentName = new MutableLiveData<>();
        }
        return currentName;
    }
}
```



**然后编写你的Activity**

```java
public class MainActivity extends AppCompatActivity {

    private NameViewModel model;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final TextView tv=findViewById(R.id.text);
        final int[] i = {100};
        findViewById(R.id.btn_1).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                CountDownTimer count=new CountDownTimer(100000,1000) {
                    @Override
                    public void onTick(long millisUntilFinished) {
//                        //postValue 工作线程调用
                        model.getCurrentName().postValue(String.valueOf(--i[0]));
//                        //必须在主线程调用
//                        model.getCurrentName().setValue("123");

                    }

                    @Override
                    public void onFinish() {

                    }
                }.start();
            }
        });


        model= ViewModelProviders.of(this).get(NameViewModel.class);

        /*
        * 写法1
        * */
//        //接收观察到的数据
//        final Observer<String> nameObserver=new Observer<String>() {
//            @Override
//            public void onChanged(String s) {
//                tv.setText(s);
//            }
//        };
//        //开始观察
//        model.getCurrentName().observe(this,nameObserver);

        //简写
        model.getCurrentName().observe(this, new Observer<String>() {
            @Override
            public void onChanged(String s) {

            }
        });

    }
}

```

