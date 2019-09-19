# Android进程优先级

## 概述

Android进程优先级分为：

- 前台进程 Foreground process
- 可见进程 Visible process
- 服务进程 Service process
- 后台进程 Background process
- 空进程  Empty process

Android中设置进程优先级方法 

```xml
<intent-filter android:priority="1000" >  
```



### 前台进程

正在与用户交互的进程，优先级最高 。

1. 进程持有一个正在和用户交互的Activity
2. 进程持有一个Service,这个Service处于以下状态：
   - Service与用户正在交互的Activity绑定。
   - Service调用了 startForeground()-设置为前台进程  ,在前台运行
   - Service执行在用户正在交互的 Activity生命周期的回调函数中。