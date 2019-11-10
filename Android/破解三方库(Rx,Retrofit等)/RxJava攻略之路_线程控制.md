# RxJava攻略之路_你还应该知道这些

通过内置的Scheduler.

## 基本概念

如果我们不指定线程，默认是在调用 subscribe 方法的线程上进行回调。如果我们想切换线程，就需要使用 Schedular.RxJava已经内置了如下5个 Scheduler.

- **Schedulers.immediate()**  

  作用于当前线程运行，相当于你在哪个线程执行代码就在哪个线程运行

- **Schedulers.newthread();**

  运行在新线程中，相当于new Thread()，每次执行都会在新线程中

- **Schedulers.io();**

  io操作，里面有一个死循环的线程池来达到最大限度的处理逻辑，虽然效率高，但是如果只是一般的计算操作，不建议放在这里，避免重复创建无用线程。

- **Schedulers.computation()**

  一些需要cpu计算的操作，比如图形，视频等。这个Scheduler使用固定线程池，大小为 CPU 核数，不要把 I/O 操作放在 compuation() 中，否则 I/O 操作的等待事件会浪费 cpu。它是 buffer,debounce,delay,inteval,sample和 skip操作符的默认调度器。

- **Schedulers.trampoline()**

  当我们想在当前线程执行一个任务时，并不是立即时，可以用 tranpoline() 将它入队。这个调度器将会处理它的队列并且按序运行队列中的每一个任务。它是 repeat 和 retry 操作符默认的调度器。

- **AndroidSchedulers.mainThread();**

  指定运行在Android主线程中



## 控制线程

Rxjava 的控制线程方法为 subscribeOn 和 observeOn ，关于这两个方法的使用，我们在操作符哪里已经学过了，这里就不再提了。



## RxJava的使用场景

关于 RxJava,最常用的莫过于它结合Retrofit，Okhttp来做网络请求框架了。

### RxJava 结合 OkHttp 访问网络

```java
private Observable<String> getObservable(){
    Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(final ObservableEmitter<String> emitter) {
            okHttpClient = new OkHttpClient();
            Request request = new Request.Builder()
                    .url("https://www.baidu.com")
                    .get()
                    .build();
            Call call = okHttpClient.newCall(request);
            call.enqueue(new Callback() {
                @Override
                public void onFailure(@NotNull Call call, @NotNull IOException e) {
                    emitter.onError(new Exception("error"));
                }

                @Override
                public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                    String str=response.body().string();
                    emitter.onNext(str);    //发送数据
                    emitter.onComplete();
                }
            });
        }
    });
    return observable;
}

private void postAsynHttp(){
    getObservable()
            .subscribeOn(Schedulers.io())		//执行在io线程
            .observeOn(AndroidSchedulers.mainThread())		//切换主线程
            .subscribe(new Observer<String>() {
                @Override
                public void onSubscribe(Disposable d) {
                    Log.e("demo","onSubscribe");
                }

                @Override
                public void onNext(String s) {
                    Toast.makeText(MainActivity.this, s, Toast.LENGTH_SHORT).show();
                }

                @Override
                public void onError(Throwable e) {
                    Log.e("demo",e.getMessage());
                }

                @Override
                public void onComplete() {
                    Log.e("demo","onComplete");
                }
            });
}
```



### RxJava 结合Retrofit

#### 更改网络请求接口

```java
/**
 * 文件下載
 * @param fileUrl
 * @return
 */
@GET
Observable<ResponseBody> download(@Url String fileUrl);
```

#### 网络请求代码如下

```java
new Retrofit.Builder()
        .baseUrl("http://101.132.64.249/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
        .build()
        .create(IpServerPost.class)
        .download("/image/1.JPG")
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Observer<ResponseBody>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(ResponseBody responseBody) {

                    // writeResponseBodyToDisk 是我写的下载保存本地工具类,可以参考一下
                    boolean toDisk = writeResponseBodyToDisk(responseBody);
                    if (toDisk) {
                        System.out.println("-----------------下载成功请查看");
                    } else {
                        System.out.println("---------------下载失败,请稍后重试");
                    }
            }

            @Override
            public void onError(Throwable e) {
                Log.e("Demo",e.getMessage());
            }

            @Override
            public void onComplete() {

            }
        });

 private boolean writeResponseBodyToDisk(ResponseBody body) {
        try {
            //判断文件夹是否存在
            File files = new File(Environment.getExternalStorageDirectory() + "/Download/");
            if (!files.exists()) {
                //不存在就创建出来
                files.mkdirs();
            }
            //创建一个文件
            File futureStudioIconFile = new File(Environment.getExternalStorageDirectory() + "/Download/ " + "download.jpg");
            //初始化输入流
            InputStream inputStream = null;
            //初始化输出流
            OutputStream outputStream = null;
            try {
                //设置每次读写的字节
                byte[] fileReader = new byte[4096];
                long fileSize = body.contentLength();
                long fileSizeDownloaded = 0;
                //请求返回的字节流
                inputStream = body.byteStream();
                //创建输出流
                outputStream = new FileOutputStream(futureStudioIconFile);
                //进行读取操作
                while (true) {
                    int read = inputStream.read(fileReader);
                    if (read == -1) {
                        break;
                    }
                    //进行写入操作
                    outputStream.write(fileReader, 0, read);
                    fileSizeDownloaded += read;
                }

                //刷新
                outputStream.flush();
                return true;
            } catch (IOException e) {
                return false;
            } finally {
                if (inputStream != null) {
                    //关闭输入流
                    inputStream.close();
                }
                if (outputStream != null) {
                    //关闭输出流
                    outputStream.close();
                }
            }
        } catch (IOException e) {
            return false;
        }
```











```
响应body：{"result":{"c":200,"m":"操作成功"},"data":{"groupChatId":"a09f81996bb145618337a6cf00ce2778","groupChatName":"一个面试机会","groupChatHeadUrl":"https://pet-im-online.oss-cn-beijing.aliyuncs.com/app/group/head/default/group_default.png","isAllowSearch":1,"isAllowInvite":1,"type":0,"notice":"","intro":"","ownerId":"5e03391a77174a9d990e4e736d2d81a8","members":[{"groupChatId":"a09f81996bb145618337a6cf00ce2778","userId":"5e03391a77174a9d990e4e736d2d81a8","userName":"1辉","userHeadUrl":"https://pet-im-online.oss-cn-beijing.aliyuncs.com/app/user/head/default/default.png","alias":"","type":2},{"groupChatId":"a09f81996bb145618337a6cf00ce2778","userId":"6e77bc23f4b5460f9cbd31e6ea69c4c5","userName":"赵海蛟","userHeadUrl":"https://pet-im-online.oss-cn-beijing.aliyuncs.com/app/user/head/default/default.png","alias":"","type":0},{"groupChatId":"a09f81996bb145618337a6cf00ce2778","userId":"be3e975e9d4540d78f3dae29c4b9ae6a","userName":"史艺辉","userHeadUrl":"https://jxl-dev.oss-cn-beijing.aliyuncs.com/app/user/head/2019/10/30/fa4be885c48c4386aab3c25149ddf9b2.jpg","alias":"","type":0},{"groupChatId":"a09f81996bb145618337a6cf00ce2778","userId":"ece60b3ae2ad4049a633a046e63a4e1c","userName":"张润潮","userHeadUrl":"https://jxl-dev.oss-cn-beijing.aliyuncs.com/app/user/head/2019/10/30/80a1c34e4b7f42e7a7561bd161ee8a5d.png?x-oss-process=image/quality,q_50/resize,l_100","alias":"","type":0},{"groupChatId":"a09f81996bb145618337a6cf00ce2778","userId":"mznxksaizuxcyxzuxxs8a7s6d7392isa","userName":"杨过","userHeadUrl":"https://jxl-dev.oss-cn-beijing.aliyuncs.com/app/user/head/2019/10/29/ec665da7b19b4087b30f60edfc028c10.png?x-oss-process=image/quality,q_50/resize,l_100","alias":"","type":0}]},"refreshTime":1572501371257}
2019-10-31 13:56:09.034 6828-7974/com.people.wpy E/PRETTY_LOGGER-okhttp: └──
```





```
userIds":["a09f81996bb145618337a6cf00ce2778","a09f81996bb145618337a6cf00ce2778","a09f81996bb145618337a6cf00ce2778","a09f81996bb145618337a6cf00ce2778","a09f81996bb145618337a6cf00ce2778"],"deptIds":[],"deviceToken":"5616fcc4f9bdb3f7a132e642fc0e4541","deviceType":"3","groupChatId":"a09f81996bb145618337a6cf00ce2778"}
2019-10-31 13:57:10.917 6828-7974/com.people.wpy E/PRETTY_LOGGER-okhttp: └────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```



