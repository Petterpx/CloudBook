# OkHttp源码解析

Demo

```java
Request request = new Request.Builder()
                .url("https://www.baidu.com")
                .get()
                .build();
        OkHttpClient okHttpClient = new OkHttpClient();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, Response response) {
                try {
                    Log.e("demo", response.body().string());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
```

#### 异步使用方法：

- 创建OkHttpClient 和 Request 对象
- 将 Request 封装成Call 对象
- 调用 Call 的 enqueue 方法进行异步请求



#### 同步使用方法：

```
Request request = new Request.Builder()
        .url("https://www.baidu.com")
        .get()
        .build();
OkHttpClient okHttpClient = new OkHttpClient();
Call call = okHttpClient.newCall(request);
try {
    call.execute().body().string();
} catch (IOException e) {
    e.printStackTrace();
}
```

- 创建OkHttpClient和 Request 对象
- 将Request封装成Call对象
- 调用 Call 的 execute() 发送请求。



## 源码解析

从Request.Builder() 方法开始

```java
 public Builder() {
      this.method = "GET";
      this.headers = new Headers.Builder();
    }
```

```java
public OkHttpClient() {
    this(new Builder());
  }
```



```java
 @Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    try {
      //核心点，添加
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain(false);
      if (result == null) throw new IOException("Canceled");
      return result;
    } finally {
      //销毁-
      
      
      client.dispatcher().finished(this);
    }
  }
```

Dipatcher 核心点

