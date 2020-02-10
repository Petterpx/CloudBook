# Android启动优化

### 为什么要做启动优化？

app启动如同我们的门面，如果第一体验都不好，用户很可能就会放弃我们这款app,如同 著名 的前端网页 8秒 定律一样。



## App启动分类

- 冷启动
- 热启动
- 温启动



### 冷启动

- 耗时最多，也是我们的衡量标准

  ![image-20200203143521351](https://tva1.sinaimg.cn/large/006tNbRwly1gbj7bndenzj30i4090dg7.jpg)



### 热启动

- 最快(即从后台到前台)

### 温启动

- 较快
- 会重走Activity的生命周期





### 经历的相关任务

#### 冷启动之前

- 启动App
- 加载空白Window
- 创建进程
- 创建Application
- 启动主线程
- 创建MainActivity
- 加载布局
- 布置屏幕
- 首帧绘制



### 优化方向

- Application和Activity生命周期，其余的部分因为我们不可控，所以我们的主要方向是在Application与Activity中。



## 启动时间测量

一般通过两种方式测量

### adb命令

```
adb shell am start -W com.people.wpy/com.people.wpy.assembly.ably_login.LoginActivity
```

```shell
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.people.wpy/.assembly.ably_login.LoginActivity }
Status: ok
Activity: com.people.wpy/.assembly.ably_login.LoginActivity
ThisTime: 1545  
TotalTime: 1545
WaitTime: 1591
Complete
```

1. ThisTime ： 最后一个Activity启动耗时
2. TotalTime:  所有Activity启动耗时
3. WaitTime:  Ams启动Activity总耗时

##### 好处与不足

- 线下使用方便，不能带到线上
- 非严谨，精确时间



### 手动打点

- 启动时埋点，启动结束埋点，二者差值

#### 优势

- 精确，可带到线上，推荐使用
- 启动时间结束位置，采用第一条数据展示





## 工具选择

- traceview
- systrace



### TraceView

- 图形的形式展现执行时间，调用栈等
- 信息全面，包含所有线程
- 可以用于埋点

#### 使用方式

- Debug.startMethodTracing("")
- Debug.stopMethodTracing()
- 生成文件在sd 卡: Android/data/packagename/files



### 缺点

- 运行时开销严重，整体都会变慢
- 可能会带偏优化方向
- Traceview 与 cpu profiler

### 

### Systrace

- 结合Android内核的数据，生成Html报告
- API18 以上，使用TraceCompat
- 轻量级，开销小
- 直观反映 cpu 利用率



#### 使用方式

- TraceCompat.beginSection("attachBaseContext");
- TraceCompat.endSection();



### cpuTime与walltime的区别

- walltime是代码执行时间
- cputime是代码消耗cpu的时间(优化参照方向)

比如 锁冲突 会造成walltime与cputime不同



## 获取方法耗时

 需要知道启动阶段所有方法耗时

- 一般通过AOP进行优化





#### Join Points

- 程序运行时的执行点，可以作为切面的地方
- 函数调用，执行
- 获取，设置变量
- 类初始化



#### PointCut

- 带条件的 JoinPoints

#### Advice

- 一种Hook,要插入代码的位置
  - Before : PointCut之前执行
  - After: PointCut 之后执行
  - Around: PointCut 之前，之后分别执行



### 语法简介

- Before : Advice ，具体插入位置

- Execution : 处理Join Point的类型，call,execution

- (* android.app.Activity.on**(..)) : 匹配规则   

  例 ： 匹配所有Activity以on开头的方法

- onActivityCalled : 要插入的代码

















 