# RxJava-倒计时功能的实现

## 方法1：

```java
 /**
     * 开启倒计时。传入间隔时间，次数
     * @param time 
     * @param sum
     */
    private void startTimer(long time, int sum) {
        Observable.interval(time, TimeUnit.MILLISECONDS)
                .take(sum)
                .map(aLong -> String.valueOf(sum - aLong))
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        disposable = d;
                    }

                    @Override
                    public void onNext(String s) {
                        //我们一般会选择将相应的时间直接设置到tv上，所以这里使用map转型
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {
                        openClick();
                        setTvCode("获取验证码");
                    }
                });      
      
 //简略写法,是不是特别简单，链式调用就是这么的爽
private void startTimer(long time,int count) {
        disposable = Observable.timer(time, TimeUnit.MILLISECONDS)
                .take(count+1)
                .map(aLong -> String.valueOf(count - aLong))
                .observeOn(AndroidSchedulers.mainThread())
                .doOnNext(this::setTvCode)
                .doOnComplete(this::openClick).subscribe();
}
 //最后不要忘记临时中断或者使用结束时取消订阅
 public void onDestroy() {
        if (disposable != null && !disposable.isDisposed()) {
            Log.e("demo", "关闭");
            disposable.dispose();
            disposable = null;
        }
    }
```



效果如下：

![Jietu20190801-120708-HD](http://ww3.sinaimg.cn/large/006tNc79ly1g5k1t168mag307u0gn1l4.gif)

