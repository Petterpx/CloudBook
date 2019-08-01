# EvenBus简单使用

### 四种线程模型

EventBus3.0有四种线程模型，分别是：

- POSTING（默认）表示事件处理函数的线程跟发布事件的线程在同一个线程。
- MAIN表示事件处理函数的线程在主线程（UI）线程，因此在这里不能进行耗时操作。
- 背景表示事件处理函数的线程在后台线程，因此不能进行UI操作。如果发布事件的线程是主线程（UI线程），那么事件处理函数将会开启一个后台线程，如果果发布事件的线程是在后台线程，那么事件处理函数就使用该线程。
- ASYNC表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作。



### 使用方式

第一步：

```java
//EvenBus
api 'org.greenrobot:eventbus:3.1.1'
```



绑定

```java
EventBus.getDefault().register(this);
```

解绑

```java
EventBus.getDefault().unregister(this);
```



Bean类

```java
package com.petterp.latte_ec.mvp;

/**
 * @author by Petterp
 * @date 2019-07-24
 */
public class MessageWrap {
    private String message;


    public MessageWrap(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```



发送消息：

```java
EventBus.getDefault().post(new MessageWrap("Petterp"));
```



接收端

```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void getKey(MessageWrap messageWrap){
    Toast.makeText(getContext(), messageWrap.getMessage(), Toast.LENGTH_SHORT).show();
}
```