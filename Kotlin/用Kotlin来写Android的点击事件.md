# 用Kotlin来写Android的点击事件

```
class MainActivity : AppCompatActivity(),View.OnClickListener{

    //实现接口
    override fun onClick(v: View?) {
        when(v?.id){
            R.id.button ->
                Toast.makeText(this,"实现接口",Toast.LENGTH_SHORT).show()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener(this)

        //匿名函数方式，最简单写法
        button.setOnClickListener {
            Toast.makeText(this,"指定onClick属性",Toast.LENGTH_SHORT).show()
        }
    }

    //指定onClick属性
    fun click(v:View?){
        Toast.makeText(this,"指定onClick属性",Toast.LENGTH_SHORT).show()
    }


}
```

