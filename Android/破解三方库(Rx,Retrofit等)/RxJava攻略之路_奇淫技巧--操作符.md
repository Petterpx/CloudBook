## 操作符入门

RxJava操作符的类型分为创建操作符，变换操作符，过滤操作符，组合操作符，错误处理操作符，福州操作符，条件和布尔操作符，算术和聚合操作符及连接操作符等，而这些操作符类型又有很多操作符，每个操作符可能还有很多变体。

### 创建操作符

#### Create()

用于创建一个被观察者

```java
public static <T> Observable<T> create(ObservableOnSubscribe<T> source)
```

**使用方式：**

```java
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
});
```

上面的代码，创建 ObservableOnSubscribe 并重写了 subscribe 方法，就可以通过 ObservableEmitter  发射器向观察者发送事件。

下面我们再来创建一个观察者，来验证这个被观察者是否创建成功。

```java
//创建观察者
Observer<String> observer=new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {
        Log.e("demo",s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        Log.e("demo","onComplete");
    }
};
//订阅
observable.subscribe(observer);
```

结果与我们最开始的例子一样，所以就用看了。



#### just()

创建一个被观察者，并发送事件，发送的事件不可以超过10个

```java
public static <T> Observable<T> just(T item)
    ...
public static <T> Observable<T> just(T item1, T item2, T item3, T item4, T item5, T item6, T item7, T item8, T item9, T item10)
```

**使用方式：**

```java
Observable.just(1,2,3)
        .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.e("demo","onSubscribe");
            }

            @Override
            public void onNext(Integer integer) {
            Log.e("demo",String.valueOf(integer));
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {
            Log.e("demo","onComplete");
            }
        });
```

上面的代码我们可以直接链式调用，结果如下：

```xml
E/demo: onSubscribe
E/demo: 1
E/demo: 2
E/demo: 3
E/demo: onComplete
```





#### interval

创建一个按固定时间间隔发射整数序列的 Observable,这个事件是从0开始，不断增1的数字。相当于定时器：

```java
public static Observable<Long> interval(long period, TimeUnit unit)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit)
```

**使用方式：**

```java
 Observable.interval(4,TimeUnit.SECONDS)
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.e("demo","onSubscribe");
                    }

                    @Override
                    public void onNext(Long aLong) {
                        Log.e("demo",""+aLong);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {
                        Log.e("demo","onComplete");
                    }
                });
```

```java
2019-07-07 10:16:24.768 19512-19512/com.petterp.test2 E/demo: onSubscribe
2019-07-07 10:16:28.775 19512-19533/com.petterp.test2 E/demo: 0
2019-07-07 10:16:32.777 19512-19533/com.petterp.test2 E/demo: 1
2019-07-07 10:16:36.775 19512-19533/com.petterp.test2 E/demo: 2
。。。
```



#### range

创建发射指定范围的整数序列 Observale ，可以拿来替代 for 循环，发射一个范围内的有序整数序列，第一个参数是起始值，并且不小于0，第二个参数为终值，左闭右开。

```java
public static Observable<Integer> range(final int start, final int count)
```

**使用方式：**

```java
Observable.range(0,5)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e("demo",String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-07 10:23:36.035 19902-19902/com.petterp.test2 E/Demo: 0
2019-07-07 10:23:36.035 19902-19902/com.petterp.test2 E/Demo: 1
2019-07-07 10:23:36.035 19902-19902/com.petterp.test2 E/Demo: 2
2019-07-07 10:23:36.035 19902-19902/com.petterp.test2 E/Demo: 3
2019-07-07 10:23:36.035 19902-19902/com.petterp.test2 E/Demo: 4
```



#### repeat

创建一个 N 次重复发射特定数据的 Observale。

```java
public static Observable<Integer> range(final int start, final int count)
```

**使用方式：**

```java
Observable.range(0,3)
        .repeat(2)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e("demo",String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-07 10:26:04.674 20018-20018/com.petterp.test2 E/demo: 0
2019-07-07 10:26:04.674 20018-20018/com.petterp.test2 E/demo: 1
2019-07-07 10:26:04.674 20018-20018/com.petterp.test2 E/demo: 2
2019-07-07 10:26:04.675 20018-20018/com.petterp.test2 E/demo: 0
2019-07-07 10:26:04.675 20018-20018/com.petterp.test2 E/demo: 1
2019-07-07 10:26:04.676 20018-20018/com.petterp.test2 E/demo: 2
```



### From操作符

#### fromArray()

和just() 类似，不过比其更加强大，可以传入多于10个的变量，并且可以传入一个数组(这里需要用包装类)。

```java
public static <T> Observable<T> fromArray(T... items）
```

**使用方式**：

```java
Integer[] sum={1,2,3,3,4,5,44,4,4,4};
//Observable.fromArray(1,2,3,3,4,5,44,4,4,4)
Observable.fromArray(sum)
        .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(Integer integer) {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
```

结果和上面都一样，所以就不用看了。



#### fromCallable()

这里的 Callable 是 java.util.concurrent 中的 Callable,Callable 和 Runnable 用法基本一致，只是它会返回一个结果值，这个结果值就是发给观察者的。

```java
public static <T> Observable<T> fromCallable(Callable<? extends T> supplier)
```

**使用方式：**

```java
Observable.fromCallable(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        return 1;
    }
}).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.e("demo",String.valueOf(integer));
    }
});
```

结果如下：

```
2019-07-07 09:24:53.569 17960-17960/com.petterp.test2 E/demo: 1
```



#### fromFuture()

```java
public static <T> Observable<T> fromFuture(Future<? extends T> future)
```

参数中的 Future 是 java.util.conncurren 中的Future,Future 的作用是增加了 cancel() 等方法操作 Callable，他可以通过 get() 方法来获取 Callable 返回的值。

**使用方式：**

```java
final FutureTask<String> futureTask=new FutureTask<>(new Callable<String>() {
    @Override
    public String call() throws Exception {
        Log.e("demo","CallableDemo is Running");
        return "返回结果";
    }
});
Observable.fromFuture(futureTask)
        .doOnSubscribe(new Consumer<Disposable>() {
            @Override
            public void accept(Disposable disposable) throws Exception {
                futureTask.run();
            }
        })
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e("demo",s);
            }
        });
```

doOnsubscribe()的作用就是只有订阅时才会发送事件。

**结果：**

```
2019-07-07 09:31:14.509 18165-18165/com.petterp.test2 E/demo: CallableDemo is Running
2019-07-07 09:31:14.510 18165-18165/com.petterp.test2 E/demo: 返回结果
```



#### fromLiterable()

用于直接发送一个 List 集合数据给观察者

```java
public static <T> Observable<T> fromIterable(Iterable<? extends T> source
```

**使用方式：**

```java
List<String> list=new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
Observable.fromIterable(list)
        .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                d.dispose();
                Log.e("demo","onSubscribe");
            }

            @Override
            public void onNext(String s) {
            Log.e("demo",s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {
                Log.e("demo","onComplete");
            }
        });
```

结果：

```java
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: onSubscribe
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: 1
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: 2
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: 3
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: 4
2019-07-07 09:42:44.592 18549-18549/com.petterp.test2 E/demo: onComplete
```



#### defer()

用于直到观察者被订阅后才会创建被观察者。

```java
public static <T> Observable<T> defer(Callable<? extends ObservableSource<? extends T>> supplier)
```

```java
//i 要定义为成员变量
i = 100;
Observable<Integer> observable = Observable.defer(new Callable<ObservableSource<? extends Integer>>() {
    @Override
    public ObservableSource<? extends Integer> call() throws Exception {
        return Observable.just(i);
    }
});
i =200;
Observer<Integer> observer = new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.e("demo","onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.e("demo",String.valueOf(integer));
    }

    @Override
    public void onError(Throwable e) {
        Log.e("demo","onError");
    }

    @Override
    public void onComplete() {
        Log.e("demo","onComplete");
    }
};
observable.subscribe(observer);
i=300;
observable.subscribe(observer);
```

**结果：**

```java
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: onSubscribe
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: 200
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: onComplete
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: onSubscribe
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: 300
2019-07-07 09:51:04.395 18801-18801/com.petterp.test2 E/demo: onComplete
```

因为defer() 只有观察者订阅的时候才会创建新的被观察者，所以每订阅一次就会打印一次，并且都是打印 i 最新值。



### 变换操作符

变换操作符的作用是对 Observale(被观察者)发射的数据按照一定规则做一些变换操作，然后将变换后的数据发射出去。变换操作符有 map,flatMap,concatMap,switchMap,flatMapIterable,buffer,groupBy,cast,window,scan等。



#### map

用于将被观察者发送的数据类型转变成其他类型。

map操作符通过制定一个 Function对象，将Observable 转换为一个 新的 Observale对象并发射，观察者收到新的 Observale 处理。假如我们要访问网络，Host 地址时常是变化的，它有时是测试服务器地址，有时可以能使正式服务器地址，但是具体界面的 URL 地址则是不变的。因此，我们可以用 map 操作符来进行转换字符操作。

```java
public final <R> Observable<R> map(Function<? super T, ? extends R> mapper)
```

**使用方法：**

```java
 final String Host="https://blog.csdn.net/";
        Observable.just(123,456).map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                return Host+integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e("demo",s);
            }
        });
```

**结果：**

```java
2019-07-07 10:46:15.508 20580-20580/com.petterp.test2 E/demo: https://blog.csdn.net/123
2019-07-07 10:46:15.509 20580-20580/com.petterp.test2 E/demo: https://blog.csdn.net/456
```



#### flatMap

```java
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)
```

这个方法可以将事件序列中的元素进行整合加工，返回一个新的被观察者

**使用方式：**

假设现在有一个 Book类与一个 BookRack类：

Book

```java
public class Book {
    private String name;
    private int money;

    public Book(String name, int money) {
        this.name = name;
        this.money = money;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }
}
```

BookRack

```java
public class BookRack {
    private int id;
    private List<Book> books=new ArrayList<>();

    public BookRack(int id, List<Book> books) {
        this.id = id;
        this.books = books;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public List<Book> getBooks() {
        return books;
    }

    public void setBooks(List<Book> books) {
        this.books = books;
    }
}
```

如果我们现在想取BookRack中全部Book的信息，我们先看用map的做法。

```java
List<Book> list=new ArrayList<>();
list.add(new Book("第一行代码",45));
list.add(new Book("Android探索艺术",45));
BookRack bookRack=new BookRack(1,list);

Observable.fromIterable(Collections.singleton(bookRack))
        .map(new Function<BookRack, List<Book>>() {
            @Override
            public List<Book> apply(BookRack bookRack) throws Exception {
                return bookRack.getBooks();
            }
        }).subscribe(new Consumer<List<Book>>() {
    @Override
    public void accept(List<Book> books) throws Exception {
        for (Book book:books){
            Log.e("demo",book.getName()+"---"+book.getMoney());
        }
    }
});
```

在accept方法里，我们还需要for循环遍历一遍。现在我们用 flatMap。

```java
List<Book> list = new ArrayList<>();
list.add(new Book("第一行代码", 45));
list.add(new Book("Android探索艺术", 45));
BookRack bookRack = new BookRack(1, list);

Observable.fromIterable(Collections.singleton(bookRack))
        .flatMap(new Function<BookRack, ObservableSource<Book>>() {
            @Override
            public ObservableSource<Book> apply(BookRack bookRack) throws Exception {
                return Observable.fromIterable(bookRack.getBooks());
            }
        })
        .subscribe(new Consumer<Book>() {
            @Override
            public void accept(Book book) throws Exception {
                Log.e("demo",book.getName()+"---"+book.getMoney());
            }
        });
```

**结果：**

```java
2019-07-07 11:16:02.353 21205-21205/com.petterp.test2 E/demo: 第一行代码---45
2019-07-07 11:16:02.353 21205-21205/com.petterp.test2 E/demo: Android探索艺术---45
```

从上面代码可以看出，flatMap() 使用起来逻辑非常清晰。



#### concatMap()

concatMap() 和 floatMap() 基本上一样的，只不过 concatMap() 转发出来的事件是有序的，而 flatMap() 是无序的。



#### flatMapIterable

flatMapIterable操作符可以将数据包装成 Iterable。

```java
public final <U> Observable<U> flatMapIterable(final Function<? super T, ? extends Iterable<? extends U>> mapper)
```

**使用方式：**

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");
list.add("6");
list.add("7");
Observable.just(list).flatMapIterable(new Function<List<String>, Iterable<String>>() {
    @Override
    public Iterable<String> apply(List<String> strings) throws Exception {
        return strings;
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.e("Demo",s);
    }
});
```

**结果：**

```java
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 1
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 2
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 3
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 4
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 5
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 6
2019-07-07 14:30:05.250 3086-3086/com.petterp.test2 E/Demo: 7
```



#### buffer

从需要发送的事件中获取一定数量的事件，并将这些事件放到缓冲区中一并执行。

其中第一个参数为缓冲区大小，第二个是跳过的个数(即指针每次向后移动的个数)

```java
public final Observable<List<T>> buffer(int count, int skip)
```

**使用方式：**

```java
Observable.just(1,2,3,4,5).buffer(2,1).concatMap(new Function<List<Integer>, ObservableSource<Integer>>() {
    @Override
    public ObservableSource<Integer> apply(List<Integer> integers) throws Exception {
        Log.e("Demo","缓冲区大小"+integers.size());
        return Observable.fromIterable(integers);
    }
}).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.e("Demo",String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 缓冲区大小2
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 1
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 2
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 缓冲区大小2
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 2
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 3
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 缓冲区大小2
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 3
2019-07-07 14:45:07.195 5002-5002/com.petterp.test2 E/Demo: 4
2019-07-07 14:45:07.196 5002-5002/com.petterp.test2 E/Demo: 缓冲区大小2
2019-07-07 14:45:07.196 5002-5002/com.petterp.test2 E/Demo: 4
2019-07-07 14:45:07.196 5002-5002/com.petterp.test2 E/Demo: 5
2019-07-07 14:45:07.196 5002-5002/com.petterp.test2 E/Demo: 缓冲区大小1
2019-07-07 14:45:07.196 5002-5002/com.petterp.test2 E/Demo: 5
```

从结果可以看出，每次发送事件，指针都会往后移动一个元素再取值，知道指针移动到没有元素的时候就会停止取值。



#### groupBy

将发送的数据进行分组，每个分组都会返回一个被观察者

```java
public final <K> Observable<GroupedObservable<K, T>> groupBy(Function<? super T, ? extends K> keySelector)
```

**使用方式：**

```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8).groupBy(new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer integer) throws Exception {
        return integer%5;
    }
}).subscribe(new Consumer<GroupedObservable<Integer, Integer>>() {
    @Override
    public void accept(final GroupedObservable<Integer, Integer> integerIntegerGroupedObservable) throws Exception {
        integerIntegerGroupedObservable.subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.e("demo","GroupedObservable onSubscribe");
            }

            @Override
            public void onNext(Integer integer) {
                Log.e("demo","GroupedObservable onNext  groupName:"+integerIntegerGroupedObservable.getKey()+"  value"+integer);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
    }
});
```

**结果：**

```java
2019-07-07 15:02:22.513 5906-5906/? E/demo: GroupedObservable onSubscribe
2019-07-07 15:02:22.513 5906-5906/? E/demo: GroupedObservable onNext  groupName:1  value1
2019-07-07 15:02:22.513 5906-5906/? E/demo: GroupedObservable onSubscribe
2019-07-07 15:02:22.515 5906-5906/? E/demo: GroupedObservable onNext  groupName:2  value2
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onSubscribe
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onNext  groupName:3  value3
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onSubscribe
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onNext  groupName:4  value4
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onSubscribe
2019-07-07 15:02:22.517 5906-5906/? E/demo: GroupedObservable onNext  groupName:0  value5
2019-07-07 15:02:22.518 5906-5906/? E/demo: GroupedObservable onNext  groupName:1  value6
2019-07-07 15:02:22.518 5906-5906/? E/demo: GroupedObservable onNext  groupName:2  value7
2019-07-07 15:02:22.518 5906-5906/? E/demo: GroupedObservable onNext  groupName:3  value8
```



### 过滤操作符

过滤操作符用于过滤和选择 Observable 发射的数据序列，让Observale 只返回满足我们条件的数据。

#### filter

filter 操作符是对圆 Observable 产生的结果自定义规则过滤，只有满足条件的结果才会提交给订阅者：

```java
public final Observable<T> filter(Predicate<? super T> predicate)
```

**使用方式：**

```java
 Observable.just(1,2,3,4,5).filter(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) {
                return integer>2;       //大于2返回并提交给订阅者
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                     Log.e("demo",String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-07 15:16:32.036 6143-6143/com.petterp.test2 E/demo: 3
2019-07-07 15:16:32.036 6143-6143/com.petterp.test2 E/demo: 4
2019-07-07 15:16:32.036 6143-6143/com.petterp.test2 E/demo: 5
```



#### elementAt

用来返回指定位置的数据，类似的有 elementAt(int ,T),其可以允许默认值。

```java
public final Maybe<T> elementAt(long index)
//第一个参数为位置，第二个为默认值
public final Single<T> elementAt(long index, T defaultItem)
```

**使用方式：**

```java
Observable.just(1,2,3,4,5).elementAt(2).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        Log.e("demo",String.valueOf(integer));
    }
});


Observable.just(1,2,3,4,5)
        //带默认值
        .elementAt(5,10)
        .subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        Log.e("demo",String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-07 15:22:08.825 6548-6548/com.petterp.test2 E/demo: 3
2019-07-07 15:22:08.839 6548-6548/com.petterp.test2 E/demo: 带默认值10
```



#### distinct

用来去重，其只允许还没有发射过的数据项通过，与distinctUntilChanged相似，其用来去掉连续重复数据。

```java
public final Observable<T> distinct()
```

```java
public final Observable<T> distinctUntilChanged()
```

**使用方式：**

```java
Log.e("demo","只允许没有发射发过的数据，即数据唯一");
Observable.just(1,1,1,2,3,2,4,5).distinct().subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
    Log.e("Demo",String.valueOf(integer));
    }
});

Log.e("demo","去掉连续重复的数据");
Observable.just(1,1,1,2,3,2,4,5).distinctUntilChanged().subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        Log.e("Demo",String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-07 15:27:47.267 6968-6968/com.petterp.test2 E/demo: 只允许没有发射发过的数据，即数据唯一
2019-07-07 15:27:47.330 6968-6968/com.petterp.test2 E/Demo: 1
2019-07-07 15:27:47.330 6968-6968/com.petterp.test2 E/Demo: 2
2019-07-07 15:27:47.331 6968-6968/com.petterp.test2 E/Demo: 3
2019-07-07 15:27:47.332 6968-6968/com.petterp.test2 E/Demo: 4
2019-07-07 15:27:47.333 6968-6968/com.petterp.test2 E/Demo: 5
2019-07-07 15:27:47.333 6968-6968/com.petterp.test2 E/demo: 去掉连续重复的数据
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 1
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 2
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 3
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 2
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 4
2019-07-07 15:27:47.336 6968-6968/com.petterp.test2 E/Demo: 5
```



#### skip,take

skip操作符将源 Observable 发射的数据过滤掉前 n 项，而 take 操作符则只取前 n 项；另外还有 skipLast 和 takeLast 操作符，则是从后面进行过滤操作。

```java
public final Observable<T> skip(long count)
```

```java
public final Observable<T> take(long count)
```

**使用方式：**

```java
Log.e("Demo","skip");
Observable.just(1,1,1,2,3,2,4,5).skip(2).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
    Log.e("Demo",String.valueOf(integer));
    }
});
Log.e("Demo","take");
Observable.just(1,1,1,2,3,2,4,5).take(2).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        Log.e("Demo",String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-07 16:35:37.188 8338-8338/com.petterp.test2 E/Demo: skip
2019-07-07 16:35:37.251 8338-8338/com.petterp.test2 E/Demo: 1
2019-07-07 16:35:37.251 8338-8338/com.petterp.test2 E/Demo: 2
2019-07-07 16:35:37.251 8338-8338/com.petterp.test2 E/Demo: 3
2019-07-07 16:35:37.251 8338-8338/com.petterp.test2 E/Demo: 2
2019-07-07 16:35:37.252 8338-8338/com.petterp.test2 E/Demo: 4
2019-07-07 16:35:37.252 8338-8338/com.petterp.test2 E/Demo: 5
2019-07-07 16:35:37.252 8338-8338/com.petterp.test2 E/Demo: take
2019-07-07 16:35:37.254 8338-8338/com.petterp.test2 E/Demo: 1
2019-07-07 16:35:37.254 8338-8338/com.petterp.test2 E/Demo: 1
```



#### ignoreElements

忽略所有源 Obsevale 产生的结果，只把 Observable 的 onCompleted 和 Error事件通知给订阅者。

```java
public final Completable ignoreElements()
```

**使用方式：**

```java
Observable.just(1,1,1,2,3,2,4,5).ignoreElements().subscribe(new CompletableObserver() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.e("demo","onSubscribe");
    }

    @Override
    public void onComplete() {
        Log.e("demo","onComplete");
    }

    @Override
    public void onError(Throwable e) {
        Log.e("demo","onError");
    }
});
```

**结果：**

```java
//订阅
2019-07-07 16:40:37.895 8682-8682/com.petterp.test2 E/demo: onSubscribe
//事件完结
2019-07-07 16:40:37.895 8682-8682/com.petterp.test2 E/demo: onComplete
```



**throttleFirst**

它会定期发射这个时间段里源 Observale 发射的第一个数据，throttleFirst 操作符默认在 computation 调度器上执行。和 throttleFirst 操作符类似的有sample 操作符，它会定时的发射源 Observale 最近发射的数据，其他的都会被过滤掉。

```java
public final Observable<T> throttleFirst(long windowDuration, TimeUnit unit) 
```

**使用方式：**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        for (int i = 0; i < 10; i++) {
            //发送事件
            emitter.onNext(i);
            Thread.sleep(100);
        }
        //事件完成
        emitter.onComplete();
    }
}).throttleFirst(200, TimeUnit.MILLISECONDS)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
            Log.e("Demo",String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-07 16:50:27.202 8932-8932/com.petterp.test2 E/Demo: 0
2019-07-07 16:50:27.407 8932-8932/com.petterp.test2 E/Demo: 2
2019-07-07 16:50:27.614 8932-8932/com.petterp.test2 E/Demo: 4
2019-07-07 16:50:27.816 8932-8932/com.petterp.test2 E/Demo: 6
2019-07-07 16:50:28.119 8932-8932/com.petterp.test2 E/Demo: 9
```



#### throtteWithTimeOut

通过时间来限流。源 Observable 每次发射出去一个数据后就会进行计时。如果在设定好的时间结束前源 Observale 有新的数据发射出来，这个数据就会被丢弃，同时 throttleWithTimeOut 重新开始计时。如果每次都是在计时结束前发射数据，那么这个限流就会走向极端；只会发射最后一个数据。其默认在 computation 调度器上执行。和 throttleWithTimeOut  操作符有类似的有 deounce 操作符，他不仅可以使用时间来过滤，还可以根据一个函数来进行限流。 

```java
 public final Observable<T> throttleWithTimeout(long timeout, TimeUnit unit) 
```

**使用方式：**

```
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        for (int i = 0; i < 10; i++) {
            //发送事件
            emitter.onNext(i);
           if (i%3==0){
               Thread.sleep(300);
           }else{
               Thread.sleep(100);
           }
        }
        //事件完成
        emitter.onComplete();
    }
}).throttleWithTimeout(200, TimeUnit.MILLISECONDS)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e("Demo", String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-07 17:01:00.964 9370-9392/com.petterp.test2 E/Demo: 0
2019-07-07 17:01:01.464 9370-9392/com.petterp.test2 E/Demo: 3
2019-07-07 17:01:01.968 9370-9392/com.petterp.test2 E/Demo: 6
2019-07-07 17:01:02.473 9370-9392/com.petterp.test2 E/Demo: 9
```

每隔100ms 发射一个数据。当发射的数据是 3 的倍数时，下一个数据就延迟到 300ms 再发射，这里设定的过滤时间时 200ms，也就是说发射间隔小于200ms的数据都会被过滤。



### 组合操作符

组合操作符可以同时处理多个 Observale 来创建我们所需要的 Observale。

#### startWith

会在源 Observale 发射的数据前面插上一些数据：

```java
public final Observable<T> startWith(T item) 
```

**使用方式:**

```java
    Observable.just(3,4,5).startWith(888).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e("demo",String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-08 09:54:39.045 13634-13634/com.petterp.test2 E/demo: 888
2019-07-08 09:54:39.045 13634-13634/com.petterp.test2 E/demo: 3
2019-07-08 09:54:39.047 13634-13634/com.petterp.test2 E/demo: 4
2019-07-08 09:54:39.048 13634-13634/com.petterp.test2 E/demo: 5
```



#### merge

merge用于将多个 Observable 合并到一个 Observable中进行发射。但是可能无序。

```JAVA
public static <T> Observable<T> merge(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2)
。。。
  public static <T> Observable<T> mergeArray(ObservableSource<? extends T>... sources)
```

**使用方式：**

```java
Observable.merge(Observable.just(1,2,3),Observable.just("Petterp")).subscribe(new Consumer<Serializable>() {
    @Override
    public void accept(Serializable serializable) throws Exception {
        Log.e("demo", String.valueOf(serializable));
    }
});
```

**結果：**

```java
2019-07-08 09:59:17.995 14139-14139/com.petterp.test2 E/demo: 1
2019-07-08 09:59:17.995 14139-14139/com.petterp.test2 E/demo: 2
2019-07-08 09:59:17.995 14139-14139/com.petterp.test2 E/demo: 3
2019-07-08 09:59:17.995 14139-14139/com.petterp.test2 E/demo: Petterp
```



#### concat

用于将多个 Observable 合并到一个 Observable中进行发射。与 merge不同的是，它是有序的。

```java
public static <T> Observable<T> concat(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2) 
    ...
```

使用方式和上面的一样，就是换了方法名而已。



#### zip

用于合并两个或者多个 Observable 发射出数据项，根据指定的函数变换它们，并发射一个新值。

```java
public static <T1, T2, R> Observable<R> zip
```

**使用方式：**

```java
Observable.zip(Observable.just(1, 2, 3), Observable.just("Petterp"), new BiFunction<Integer, String, String>() {
    @Override
    public String apply(Integer integer, String s) {
        return s+integer;
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) {
        Log.e("demo",s);
    }
});
```

**结果：**

```java
2019-07-08 10:13:40.711 14690-14690/com.petterp.test2 E/demo: Petterp1
```

可以发现，当两个 Observable 参数个数不同时，只发送可以匹配上的。



#### combineLastest&combineLatestDelayError()

它的作用与 zip类似，但是 comBineLastest() 发送事件的序列是与 发送的时间线有关的，当 combineLatest() 中所有的 Observable 都发送了事件，只要其中有一个 Observable 发送事件，这个事件就会和其他 Observable 最近发送的时间结合起来发送。

```java
public static <T1, T2, R> Observable<R> combineLatest
。。。
```

**使用方式：**

```java
Observable.combineLatest(
       Observable.intervalRange(1,4,1,1,TimeUnit.SECONDS)
               .map(new Function<Long, String>() {
                   @Override
                   public String apply(Long aLong) {
                       String s1="A"+aLong;
                       Log.e("Demo","---------A发送的事件"+s1);
                       return "A"+aLong;
                   }
               }),
                Observable.intervalRange(1,5,2,2,TimeUnit.SECONDS)
                .map(new Function<Long, String>() {
                    @Override
                    public String apply(Long aLong) {
                        String s2="B"+aLong;
                        Log.e("Demo","---------B发送的事件"+s2);
                        return "B"+aLong;
                    }
                }),
        new BiFunction<String, String, String>() {
    @Override
    public String apply(String s1, String s2) {
        return s1+s2;
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) {
        Log.e("demo","接收到的事件"+s);
    }
});
```

**结果：**

```java
2019-07-08 10:33:18.305 15053-15083/com.petterp.test2 E/Demo: -------------A发送的事件A1
2019-07-08 10:33:19.305 15053-15084/com.petterp.test2 E/Demo: ------------B发送的事件B1
2019-07-08 10:33:19.305 15053-15083/com.petterp.test2 E/Demo: -------------A发送的事件A2
2019-07-08 10:33:19.305 15053-15084/com.petterp.test2 E/demo: ----------------------------接收到的事件A1B1
2019-07-08 10:33:19.305 15053-15084/com.petterp.test2 E/demo: ----------------------------接收到的事件A2B1
2019-07-08 10:33:20.304 15053-15083/com.petterp.test2 E/Demo: -------------A发送的事件A3
2019-07-08 10:33:20.304 15053-15083/com.petterp.test2 E/demo: ----------------------------接收到的事件A3B1
2019-07-08 10:33:21.305 15053-15083/com.petterp.test2 E/Demo: -------------A发送的事件A4
2019-07-08 10:33:21.305 15053-15084/com.petterp.test2 E/Demo: ------------B发送的事件B2
2019-07-08 10:33:21.305 15053-15083/com.petterp.test2 E/demo: ----------------------------接收到的事件A4B1
2019-07-08 10:33:21.305 15053-15083/com.petterp.test2 E/demo: ----------------------------接收到的事件A4B2
2019-07-08 10:33:23.305 15053-15084/com.petterp.test2 E/Demo: ------------B发送的事件B3
2019-07-08 10:33:23.305 15053-15084/com.petterp.test2 E/demo: ----------------------------接收到的事件A4B3
2019-07-08 10:33:25.307 15053-15084/com.petterp.test2 E/Demo: ------------B发送的事件B4
2019-07-08 10:33:25.308 15053-15084/com.petterp.test2 E/demo: ----------------------------接收到的事件A4B4
2019-07-08 10:33:27.305 15053-15084/com.petterp.test2 E/Demo: ------------B发送的事件B5
2019-07-08 10:33:27.306 15053-15084/com.petterp.test2 E/demo: ----------------------------接收到的事件A4B5
```

从上面我们可以看出，每隔1秒A发送一次事件，每隔两秒B发送一次事件，当A发送事件后，只要另一个B发送，此时combineLatest 就会将A与最近的B事件合成然后发送。所以，当A发送事件后，如果B没有发送，那么根本就不会发生结合，也就是被观察者发送事件后，会与其他被观察者最近发送事件结合起来发送，如果没有则不发送。

combineLatestDelayError()相比上面多了延时发送，所以差别使用方式也并无区别。



### 辅助操作符

#### delay

用于让原始 Observable 在发射每项数据之前都暂停一段指定的时间段。

```java
public final Observable<T> delay(long delay, TimeUnit unit) 
```

**使用方式：**

```java
Observable.just(1,2,3).delay(2,TimeUnit.SECONDS).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.e("Demo",String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-08 11:03:10.386 15792-15792/com.petterp.test2 E/Demo: onCreate
2019-07-08 11:03:12.533 15792-15824/com.petterp.test2 E/Demo: 1
2019-07-08 11:03:12.533 15792-15824/com.petterp.test2 E/Demo: 2
2019-07-08 11:03:12.533 15792-15824/com.petterp.test2 E/Demo: 3
```

从上面结果我们可以发现，数据延迟2秒之后才发送的。



#### Do

Do系列操作符就是为原始 Observable 的生命周期事件注册一个回调，当 Observable 的某个事件发生时就会调用这些回调。

- doOnEach:为 Observable 注册这样一个回调，当 Observable 每发射一项数据时就会调用它一次，包括onNext,onError 和 onCompleted.
- doOnNext: 只有执行 onNext 的时候会被调用。
- doOnSubscribe：当观察者订阅 Observable 时就会被调用。
- doOnUnsubscribe: 当观察者取消订阅 Observable 时就会被调用；Observable 通过 onError 或者 onCompleted 结束时，会取消订阅所有的 Subscriber.
- doOnCompleted：当Observable 这么好穿终止调用 onCompleted 时会被调用。
- doOnError: 当Observable 异常终止调用 onError 时会被调用。
- doOnTerminate: 当Observable 终止(无论是正常终止还时异常终止)之前会被调用。
- doFinally: 当Observable 终止(无论是正常终止还时异常终止)之后会被调用。

我们用 doOnError 来举例看一下：

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onError(new Throwable("终止"));
        emitter.onNext(3);
        emitter.onComplete();
    }
})
        .doOnError(new Consumer<Throwable>() {
           @Override
           public void accept(Throwable throwable) {
               Log.e("Demo", String.valueOf(throwable));
           }
       })
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e("demo", String.valueOf(integer));
            }
        });
```

**结果：**

```java
2019-07-08 11:26:29.410 16304-16304/com.petterp.test2 E/demo: 1
2019-07-08 11:26:29.410 16304-16304/com.petterp.test2 E/demo: 2
2019-07-08 11:26:29.411 16304-16304/com.petterp.test2 E/Demo: java.lang.Throwable: 终止
```



#### subscribeOn,observeOn

指定  被观察者 自身在那个线程上运行。如果多次调用，只有第一次有效。

observeOn 用来指定 Observer 所运行的线程，也就是发射出的数据在那个线程上使用。一般情况下指定在主线程运行，便于更新ui.

学习之前，我们先了解一些基本的函数含义：

- **Schedulers.immediate()**  

  作用于当前线程运行，相当于你在哪个线程执行代码就在哪个线程运行

- **Schedulers.newthread();**

  运行在新线程中，相当于new Thread()，每次执行都会在新线程中

- **Schedulers.io();**

  io操作，里面有一个死循环的线程池来达到最大限度的处理逻辑，虽然效率高，但是如果只是一般的计算操作，不建议放在这里，避免重复创建无用线程。

- **Schedulers.computation()**

  一些需要cpu计算的操作，比如图形，视频等

- **AndroidSchedulers.mainThread();**

  指定运行在Android主线程中

**使用方式：**

```java
Log.e("demo","当前线程"+Thread.currentThread().getName());
Observable.just(1,2,3,4)
        .subscribeOn(Schedulers.newThread())        //创建新线程
        .subscribeOn(AndroidSchedulers.mainThread())  //在主线程运行
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                Log.e("demo",Thread.currentThread().getName());
            }
        });
```

**结果：**

```java
2019-07-08 14:41:48.437 19763-19763/com.petterp.test2 E/demo: 当前线程main
2019-07-08 14:41:48.570 19763-19785/com.petterp.test2 E/demo: RxNewThreadScheduler-1
2019-07-08 14:41:48.570 19763-19785/com.petterp.test2 E/demo: RxNewThreadScheduler-1
```

在这里例子里，我们先打印当前所在线程，然后创建子线程，最后又切换回主线程，可以发现结果还是在子线程，这就证明，当多次执行subscribeOn方法，结果只会执行第一个。



##### 接着看看 observeOn()

**使用方式：**

```java
Observable.just(1,2,3,4)
        .observeOn(Schedulers.newThread())        //创建子线程
        .map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) {
                Log.e("demo","map-----"+Thread.currentThread().getName());
                return String.valueOf(integer);
            }
        })
        .observeOn(AndroidSchedulers.mainThread())      //切换主线程
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) {
                Log.e("demo","观察者:"+s+"---"
                        +Thread.currentThread().getName());
            }
        });
```

**结果：**

```java
2019-07-08 14:50:22.231 20039-20068/com.petterp.test2 E/demo: map-----RxNewThreadScheduler-1
2019-07-08 14:50:22.232 20039-20068/com.petterp.test2 E/demo: map-----RxNewThreadScheduler-1
2019-07-08 14:50:22.307 20039-20039/com.petterp.test2 E/demo: 观察者:1---main
2019-07-08 14:50:22.307 20039-20039/com.petterp.test2 E/demo: 观察者:2---main
2019-07-08 14:50:22.307 20039-20039/com.petterp.test2 E/demo: 观察者:3---main
2019-07-08 14:50:22.307 20039-20039/com.petterp.test2 E/demo: 观察者:4---main
```

可以看到每指定一次线程，就会生效一次。



#### timeout

如果原始 OBservable 过了指定一端时长没有发射任何数据，timeout操作符 会以一个 onError 通知终止这个 Observable ,或者继续执行一个 备用的 Observable .

```java
public final Observable<T> timeout(long timeout, TimeUnit timeUnit, ObservableSource<? extends T> other) 
    。。。
```

**使用方式：**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; i < 4; i++) {
                    Thread.sleep(i * 100);
                    emitter.onNext(i);
                }
                emitter.onComplete();
            }
        })
                .timeout(250, TimeUnit.MILLISECONDS, Observable.just(888))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e("demo",String.valueOf(integer));
                    }
                });
```

**结果：**

```java
2019-07-08 15:03:59.529 20849-20849/com.petterp.test2 E/demo: 0
2019-07-08 15:03:59.630 20849-20849/com.petterp.test2 E/demo: 1
2019-07-08 15:03:59.831 20849-20849/com.petterp.test2 E/demo: 2
2019-07-08 15:04:00.083 20849-20870/com.petterp.test2 E/demo: 888
```

如果 Observable 在250ms这段时长没有发射数据，就会切换到 Observable.just(888).



### 错误处理操作符

Rx在错误出现的时候就会调用 Subscriber 的onError 方法将错误分发出去，由Subscriber 自己处理错误。但是如果每个 Subscriber 都处理一遍的话，工作量就有点大了，这时候可以使用错误处理符。



#### catch

catch操作符拦截原始 Observable 的onError 通知，将它替换为其他数据项或数据序列，让产生的 Observable 能够正常终止或者根本不终止。Rx将catch实现为以下3个不同的操作符。

- onErrotRetum: Observable遇到错误是返回原有 Observable 行为的备用 Observable，备用 Observable 会忽略原有 Observable 的 onError 调用，不会将错误传递给观察者。作为替代，他会发射一个特殊的项并调用观察者的 onComplete方法。

- onErrorResumeNext: Observable遇到错误时返回原有 Observable ，备用 Observable会忽略原有 Observable 的 onError调用，不会将错误传递给观察者。作为替代，它会发射一个特殊的项并调用观察者的 onCompleted 方法。

- onExceptionResumeNext: 它和 onErrorResumeNext 类似。不同的是，如果 onError 收到的 Throwable 不是 一个 Exception,它会将错误传递给观察者的 onError方法，不会使用备用的 Observable.

  

**使用方式：**(使用onErrorReturn)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        for (int i=0;i<5;i++){
            if (i>2){
                emitter.onError(new Throwable("Throwable"));
            }
            emitter.onNext(i);
        }
        emitter.onComplete();
    }
}).onErrorReturn(new Function<Throwable, Integer>() {
    @Override
    public Integer apply(Throwable throwable) throws Exception {
        return 6;
    }
}).subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.e("Demo","onNext:"+integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.e("Demo",e.getMessage());
    }

    @Override
    public void onComplete() {
        Log.e("Demo","onComplete");
    }
});
```

**结果：**

```
2019-07-08 15:31:51.416 21336-21336/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:31:51.416 21336-21336/com.petterp.test2 E/Demo: onNext:1
2019-07-08 15:31:51.416 21336-21336/com.petterp.test2 E/Demo: onNext:2
2019-07-08 15:31:51.416 21336-21336/com.petterp.test2 E/Demo: onNext:6
2019-07-08 15:31:51.416 21336-21336/com.petterp.test2 E/Demo: onComplete
```



#### retry

retry不会将原始 Observable 的 onError 通知传递给观察者，它会订阅这个 Observable，再给它一次机会无错误的完成其数据结构。retry 总是传递 onNext 通知给观察者，由于重写订阅，这可能会造成数据项重复。Rx 中的实现为 retry和 retryWhen.

这里我们使用retry（Long）来举例

**使用方式：**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        for (int i=0;i<5;i++){
            if (i==1){
                emitter.onError(new Throwable("Throwable"));
            }else{
                emitter.onNext(i);
            }
        }
        emitter.onComplete();
    }
}).retry(5).subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.e("Demo","onNext:"+integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.e("Demo",e.getMessage());
    }

    @Override
    public void onComplete() {
        Log.e("Demo","onComplete");
    }
});
```

**结果：**

```java
2019-07-08 15:51:28.556 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.557 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.557 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.557 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.557 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.558 17051-17051/com.petterp.test2 E/Demo: onNext:0
2019-07-08 15:51:28.558 17051-17051/com.petterp.test2 E/Demo: Throwable
```

可以发现，在重复尝试订阅5次之后，它发送了onError通知观察者。



#### retryWhen

当被观察者接收到异常或者错误事件时会回调该方法，这个方法会返回一个新的被观察者。如果返回的被观察者发送 Error 事件则之前的被观察者不会继续发送事件，如果发送正常事件则之前的被观察者会继续不断重试发送事件。

**使用方式：**

```java
 Observable.create(new ObservableOnSubscribe<Integer>() {
        @Override
        public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
            for (int i=0;i<5;i++){
                if (i==2){
                    emitter.onError(new Exception("404"));
                }else{
                    emitter.onNext(i);
                }
            }
            emitter.onComplete();
        }
    }).retryWhen(new Function<Observable<Throwable>, ObservableSource<?>>() {
        @Override
        public ObservableSource<?> apply(Observable<Throwable> throwableObservable) throws Exception {
            return throwableObservable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                @Override
                public ObservableSource<?> apply(Throwable throwable) throws Exception {
                    if (throwable.toString().equals("java.lang.Exception: 404")){
                        return Observable.error(new Throwable("终止"));
                    }else{
                        return Observable.just("没事");
                    }
                }
            });
        }
    }).subscribe(new Observer<Integer>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onNext(Integer integer) {
            Log.e("Demo","onNext:"+integer);
        }

        @Override
        public void onError(Throwable e) {
            Log.e("Demo",e.getMessage());
        }

        @Override
        public void onComplete() {
            Log.e("Demo","onComplete");
        }
    });
}
```

**结果：**

```java
2019-07-08 16:13:12.598 19771-19771/com.petterp.test2 E/Demo: onNext:0
2019-07-08 16:13:12.598 19771-19771/com.petterp.test2 E/Demo: onNext:1
2019-07-08 16:13:12.609 19771-19771/com.petterp.test2 E/Demo: 终止
```

注意：如果未返回error事件，则会不断重试发送事件。





### 布尔操作符

布尔操作符有 all,contains,isEmpty,exists，和 sequeceEqual

#### all

根据一个函数对源 Observable 发射的所有数据进行判断，最终返回的结果就是这个判断结果。这个函数使用发射的数据作为参数，内部判断所有的数据是否满足我们定义好的判断条件。如果全部都满足则返回 true，否则就返回 false。

```java
public final Single<Boolean> all(Predicate<? super T> predicate)
```

**使用方式：**

```java
Observable.just(1,2,3,4)
        .all(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer<3;
            }
        })
        .subscribe(new Consumer<Boolean>() {
            @Override
            public void accept(Boolean aBoolean) throws Exception {
                Log.e("demo","结果"+aBoolean);
            }
        });
```

**结果：**

```java
2019-07-08 16:21:14.713 19999-19999/? E/demo: 结果false
```



#### contains，isEmpty

**contains**: 用于判断源  Observable 所发射的数据是否包含某一个数据。如果包含该数据，会返回 true.如果源 Observable 已经结束了 却还没有发射这个数据，则返回false。 

**isEmpty:** 用来判断源 Observable 是否发射过数据。如果发射过该数据，就会返 false.否则true.

```java
 public final Single<Boolean> contains(final Object element)
 public final Single<Boolean> isEmpty() 
```

**使用方式：**

```java
Observable.just(1,2,3,4)
        .contains(1)
        .subscribe(new Consumer<Boolean>() {
            @Override
            public void accept(Boolean aBoolean) throws Exception {
                Log.e("demo","结果"+aBoolean);
            }
        });

Observable.just(1,2,3,4)
                .isEmpty()
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean aBoolean) throws Exception {
                        Log.e("demo","结果"+aBoolean);
                    }
                });
```

**结果：**

```java
2019-07-08 16:30:58.382 20310-20310/? E/demo: 结果true
2019-07-08 16:30:58.383 20310-20310/? E/demo: 结果false
```



### 条件操作符

条件操作符有 amb,defaultEmpty ,skipUntil, skipWhile,takeUntil 和 takeWhile 等。

#### amb

用于对给定两个或多个 Observale,它只发射 首先发射数据或通知的那个 Observable 的所有数据

```java
  public static <T> Observable<T> amb(Iterable<? extends ObservableSource<? extends T>> sources) 
```

**使用方式：**

```java
ArrayList<Observable<Long>> list=new ArrayList<>();
list.add(Observable.intervalRange(1,5,2,1,TimeUnit.SECONDS));
list.add(Observable.intervalRange(6,5,3,1,TimeUnit.SECONDS));
Observable.amb(list).subscribe(new Consumer<Long>() {
    @Override
    public void accept(Long aLong) {
        Log.e("Demo", String.valueOf(aLong));
    }
});
```

**結果**：

```java
2019-07-08 16:45:34.584 20815-20833/com.petterp.test2 E/Demo: 1
2019-07-08 16:45:35.584 20815-20833/com.petterp.test2 E/Demo: 2
2019-07-08 16:45:36.584 20815-20833/com.petterp.test2 E/Demo: 3
2019-07-08 16:45:37.584 20815-20833/com.petterp.test2 E/Demo: 4
2019-07-08 16:45:38.585 20815-20833/com.petterp.test2 E/Demo: 5
```



#### defaultEmpty

如果原始 Observable 没有发送数据，就发射一个默认数据。

```java
public final Observable<T> defaultIfEmpty(T defaultItem)
```

**使用方式：**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) {
        emitter.onComplete();    //未发送数据
    }
})
        .defaultIfEmpty(3).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        Log.e("Demo", String.valueOf(integer));
    }
});
```

**结果：**

```java
2019-07-08 16:48:38.773 21026-21026/com.petterp.test2 E/Demo: 3
```



### 转换操作符

转换操作符用来将 Observable 转换为另一种对象或数据结构。转换操作符有 toList,toSortedList,toMap,toMultiMap,getIterator和 nest。

#### toList

用于将多项数据且为每一项数据调用 onNext 方法的 Observable 发射的多项数据组成一个 List，然后调用 一次 onNext 方法传递整个列表。

```java
public final Single<List<T>> toList()
```

**使用方式：**

```java
Observable.just(1,2,3).toList().subscribe(new Consumer<List<Integer>>() {
    @Override
    public void accept(List<Integer> integers) throws Exception {
        Log.e("Demo", String.valueOf(integers));
    }
});
```

**结果：**

```java
2019-07-08 16:57:51.360 21302-21302/? E/Demo: [1, 2, 3]
```



#### toSortList

类似于toList操作符，不同的是，它会对产生的列表排序，默认是自然升序，如果发射的数据项没有实现 Comparable 接口，会抛出一个异常。当然，若发射的数据项没有实现 Comparable 接口，可以使用 toSortedList(Fon)变体。

```java
 public final Single<List<T>> toSortedList
```

**使用方式：**

```java
Observable.just(2,8,3).toSortedList().subscribe(new Consumer<List<Integer>>() {
    @Override
    public void accept(List<Integer> integers) throws Exception {
        Log.e("Demo", String.valueOf(integers));
    }
});
```

**结果：**

```java
2019-07-08 17:02:27.166 21549-21549/com.petterp.test2 E/demo: [2, 3, 8]
```



#### toMap

用于收集原始 Observable 发射的所有数据项到一个 Map(默认HashMap),然后发射这个 Map.可以提供一个用于生成 Map的 key的函数，也可以提供一个 函数转换数据项到 Map 存储的值(默认数据项本身就是值)。

```java
public final <K> Single<Map<K, T>> toMap(final Function<? super T, ? extends K> keySelector)
```

**使用方式：**

```java
Test.class
class Test {
        private String key;
        private String name;

        public Test(String key, String name) {
            this.key = key;
            this.name = name;
        }

        public String getKey() {
            return key;
        }

        public String getName() {
            return name;
        }
    }

Test test1 = new Test("1001", "Petterp");
Test test2 = new Test("0003", "Mito");
Observable.just(test1, test2).toMap(new Function<Test, String>() {
    @Override
    public String apply(Test test) throws Exception {
        return test.getKey();	//根据Test的key作为key值
    }
}).subscribe(new Consumer<Map<String, Test>>() {
    @Override
    public void accept(Map<String, Test> stringTestMap) throws Exception {
        Log.e("demo", stringTestMap.get("1001").getName());
    }
});
```

**结果：**

```java
2019-07-08 17:14:20.136 22010-22010/com.petterp.test2 E/demo: Petterp
```

