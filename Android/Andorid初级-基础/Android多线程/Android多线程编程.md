# Android多线程编程

> Android沿用了Java的线程模型，一个Android应用在创建的时候会开启一个线程，我们叫它主线程或者UI线程。
>
> 如果我们想要访问网络或者数据库等耗时操作，都会开启子线程去处理，从 Android3.0 开始，系统要求网络访问必须在子线程中进行，否则会抛出异常；也就是为了避免主线程被耗时操作阻塞从而产生 ANR。

### 线程基础

#### 进程与线程

##### 什么是进程？

> 进程是操作系统结构的基础，是程序在一个数据集合上运行的过程，是系统进行资源分配和调度的基本单位。进程可以看做是程序的实体，同时，他也是线程的容器。

##### 什么是线程？

> 线程是操作系统调度的最小单元，也叫轻量级进程。在一个进程中可以创建多个线程，这些线程都拥有各自的计数器，堆栈和局部变量等属性，并且能够访问共享的内存变量。

##### 为什么要使用多线程？

> 在操作系统级别上来看主要有以下几个方面：
>
> - 使用多线程可以减少程序的响应时间。
> - 与进程相比，线程的创建和切换开销更小，同时多线程在数据共享方面效率非常高。
> - 使用多线程能简化程序的结构，使程序便于理解和维护。

#### 线程的状态

Java的线程运行的声明周期中可能会处于6中不同的状态。

> - New 新创建状态。线程被创建，还没有调用Start方法，在线程运行之前还有一些基础工作要做。
> - Runnable  可运行状态。一旦调用start方法，线程就处于 Runnable状态。一个可运行的线程可能正在运行也可能没有运行，这取决于操作系统给线程提供运行的时间。
> - Blocked  阻塞状态。表示线程被锁阻塞，它暂时不活动。
> - Waiting 等待状态，线程暂时不活动，并且不运行任何代码，这消耗最少的资源，直到线程调度器重新激活它。
> - Timed waiting  超时等待状态。和等待状态不同的是，它是可以在指定的时间自行返回的。
> - Terminated  终止状态。 表示当前线程已经执行完毕。导致线程终止有两种情况： 第一种就是run方法执行完毕正常退出；第二种就是因为没有一个捕获的异常而终止了 run方法，导致线程进入了终止状态。

![1551509496252](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1551509496252.png)

> 线程创建后，调用Thread 的 Start方法，开始进入运行状态，当线程执行 wait 方法后，线程进入等待状态，进入等待状态的线程需要其他线程通知才能返回运行状态。超时等待相当于在等待状态加上了时间限制，如果超过时间限制，则线程返回运行状态。当线程调用到同步方法时，如果线程没有获得所则进入阻塞状态，当阻塞状态的线程获取到锁是则重新回到运行状态。当线程执行完毕或者遇到以外异常终止时，都会进入终止状态。

#### 创建线程

1. 继承Thread类，重写run方法

   > Thrad本质上也是实现了 Runnable接口的一个实例。需要注意的是调用 start方法后并不是立即执行多线程的代码，而是使该线程变为可运行状态，什么时候运行多线程代码是否操作系统决定的。
   >
   > ```java
   > public class MyClass extends Thread{
   >     @Override
   >     public void run() {
   >         System.out.println("Petterp");
   >     }
   > 
   >     public static void main(String[] args) {
   >         MyClass myClass=new MyClass();
   >         myClass.start();
   >     }
   > }
   > ```

2. 实现Runnable接口，并实现该接口的run方法

   ```java
   public class MyClass extends Thread{
       @Override
       public void run() {
           System.out.println("Petterp");
       }
   
       public static void main(String[] args) {
           MyClass myClass=new MyClass();
           myClass.start();
       }
   }
   
   ```

3. 实现Callable接口，重写call方法

   > Callable接口是属于Expecutor框架中的功能类，Callable接口与Runnable接口的功能类似，但提供了比Runnable更强大的功能。
   >
   > - Callable 可以在任务接受后提供一个返回值，Runnable无法提供这个功能。
   > - Callable 中的call方法可以抛出异常，而Runnable的fun方法不能抛出异常。
   > - 运行Callable 可以拿到一个 Future的对象，Future对象表示异步计算得到的结果，他提供了检查计算是否完成的方法。由于线程属于异步计算模型，因此无法从别的线程中得到函数的返回值，在这种情况下就可以使用 Future 来监视目标线程调用 call 方法的情况。但调用 Future的 get() 方法以获取结果是，当前线程就会阻塞，直到 call 方法返回结果。
   >
   > ```java
   > class TestCallable implements Callable{
   >         public Object call() throws Exception {
   >             Thread.sleep(1000);
   >             return "Petterp";
   >         }
   >     }
   > 
   > 
   > public class MyClass {
   >     public static void main(String[] args) {
   >         TestCallable testCallable=new TestCallable();
   >         ExecutorService service= Executors.newCachedThreadPool();
   >         Future future=service.submit(testCallable);
   >         try {
   >             //等待线程结束，并返回结果
   >             System.out.println(future.get());
   >         } catch (InterruptedException e) {
   >             e.printStackTrace();
   >         } catch (ExecutionException e) {
   >             e.printStackTrace();
   >         }
   >     }
   > }
   > ```



   #### 理解中断

   > 当线程的run方法执行完毕，或者在方法中出现没有捕获的异常时，线程将终止。在Java早期版本中有一个Stop方法，其他线程可以调用它终止线程，但是这个方法现在已经被弃用了。interrupt 方法可以用来请求中断线程。当一个线程调用 interrupt 方法时，线程的中断标识位将被置位(中断标识位为 true),线程会不时的检测这个中断标识位，以判断线程是否应该被中断。要想知道线程是否被置位，可以调用Thread.currentThread().inInterrupted() 
   >
   > ```
   > while(!Thread.currentThread().isIntterrupted()){
   >     
   > }
   > ```
   >
   >
   >
   > 还可以调用Thread.interrupted() 来对中断标识位进行复位。但是如果一个线程被阻塞，就无法检测中断状态。如果一个线程处于阻塞状态，线程在检查中断标识符是如果发现中断标识位为 true，则会在阻塞方法调用处抛出 InterruptedException 异常，并且在抛出异常前将线程的中断标识位复位，即重新设置为false，需要注意的是被中断的线程不一定会终止，中断线程是为了引起线程的注意，被中断的线程可以决定如何去响应中断，如果是比较重要的线程则不会理会中断，而大部分情况则是线程会将中断作为一个终止的请求。另外，不要在底层代码里捕获 InterruptedExcepetion 异常后不做处理：如下图所示
   >
   > ```java
   > class TestCallable extends Thread{
   >     @Override
   >     public  void run() {
   >                 try {
   >                     Thread.sleep(5000);
   >                 } catch (InterruptedException e) {
   >                     e.printStackTrace();
   >                 }
   >             }
   >     }
   > ```
   >
   > 可以使用以下两种方式处理异常：
   >
   > 1. 在catch 语句中，调用Thread.currentThread.interrupt() 来设置中断状态（因为抛出异常后中断标识符为复位），让外界通过判断 Thread.currentThread().isInterrupted()来决定是否终止线程还是继续下去。
   >
   >    ```java
   >    class TestCallable extends Thread{
   >        @Override
   >        public  void run() {
   >                    try {
   >                        Thread.sleep(5000);
   >                    } catch (InterruptedException e) {
   >                        interrupted();
   >                    }
   >                }
   >        }
   >    ```
   >
   > 2. 更好的做法就是，不适用try来捕获这样的异常，让方法直接抛出，这样调用者可以捕获这个异常，如下
   >
   >    ```java
   >    class TestCallable extends Thread {
   >        @Override
   >        public void run() {
   >    
   >        }
   >        void myTask() throws InterruptedException {
   >            sleep(5000);
   >        }
   >    }
   >    ```

---

   #### 安全的终止线程

```java
import java.util.concurrent.TimeUnit;

class TestCallable extends Thread {
    int i=0;
    @Override
    public void run() {
        System.out.println(i++);
    }
}



public class MyClass {
    public static void main(String[] args) throws InterruptedException {
        final TestCallable tes=new TestCallable();
        tes.start();
        TimeUnit.MILLISECONDS.sleep(10);
        tes.interrupt();
    }
}

```

```java
class TestCallable extends Thread {
    int i=0;
    private volatile  boolean on=true;
    @Override
    public void run() {
        while (on){
            System.out.println(i++);
        }
        System.out.println("stop");
    }
    public void cancel(){
        on=false;
    }
}



public class MyClass {
    public static void main(String[] args) throws InterruptedException {
        final TestCallable tes=new TestCallable();
        tes.start();
        TimeUnit.MILLISECONDS.sleep(100);
        tes.cancel();
    }
}
```



---

### 同步

> 在多线程应用中，两个或两个以上的线程需要共享对同一个数据的存取。如果两个线程存取相同的对象，并且每一个线程都调用了修改该对象的方法，这种情况通常被称为竞争条件。而解决这种问题的办法通常是当线程A调用修改对象方法时，我们就交给它一把锁，等他处理完后在把锁给另一个要调用这个方法的线程。

#### 重入锁和条件对象

> synchronized 关键字提供了锁以及相关的条件。大多数需要显示锁的情况使用 synchronized 非常方便，但是等我们了解重入锁和条件对象时，能更好的理解 synchronized 关键字。重入锁 ReentrantLock 是Java se5.0引入的，就是支持重进入的锁，他表示锁能够支持一个线程对资源的重复加锁。
>
> ```java
>  Lock  lock = new ReentrantLock();
>         try {
>             ...
>         }catch (Exception e){
>             lock.unlock();
>         }
> ```
>
> Demo如下
>
> ```java
> class Test implements Runnable {
>     private Boolean on;
> 
>     public Test(Boolean on) {
>         this.on = on;
>     }
> 
>     @Override
>     public void run() {
>         new MyClass().getData(on);
>     }
> }
> 
> public class MyClass {
>     private static Lock lock;
>     private static Condition condition;
> 
>     public static void main(String[] args) {
>         lock = new ReentrantLock();
>         
>         //我们这个线程已经获取了锁，具有排他性，别的线程无法获取锁，此时我们需要引入条件对象
>         //一个锁对象拥有多个相关的条件对象。
>         //得到条件对象
>         condition = lock.newCondition();
>         Thread a = new Thread(new Test(true));
>         Thread b = new Thread(new Test(false));
>         a.start();
>         b.start();
>     }
> 
>     void getData(Boolean on) {
>         lock.lock();
>         System.out.println("我被锁住了");
>         try {
>             if (on) {
>                 System.out.println("我阻塞了,放弃锁");
>                 condition.await();
>             }
>              condition.signalAll();
>             System.out.println("解除阻塞");
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         } finally {
>             lock.unlock();
>         }
>     }
> }
> ```
>
> 一旦一个线程调用await 方法，他就会进入该条件的等待集并处于阻塞状态，直到另一个线程调用了同一个条件的 signalAll 方法时为止。
>
> 当调用 singalAll 方法时并不是立即激活一个等待线程，他仅仅解除了等待线程的阻塞，以便这些线程能够在当前线程退出同步方法后，通过竞争实现对对象的访问。还有一个方法时 sinal ，它则是随机解除某个线程的足改，如果该线程任然不能运行，则再次被阻塞。 如果没有其他线程再次调用 singal ，那么系统就死锁了

---

#### 同步方法

> Java 的每一个对象都有一个内部锁，如果一个方法用 synchronized 关键字声明，那么对象的锁将保护整个方法。也就是说，要调用该方法，线程必须获得内部的对象锁。
>
> ```java
>  public synchronized  void method(){
>         
>     }
>     
>     等价于下面这个Demo
>     
>     Lock lock=new ReentrantLock();
>     public void method(){
>         try {
>             ...
>         }finally {
>             lock.unlock();
>         }
>     }
> ```
>
> 对于上面的例子，我们可以用 synchronized 修饰getData ，而不是使用一个显示锁。内部对象锁只有一个相关条件， wait 方法将一个线程添加到等待集中， notifyAll 或者 notify 方法解除等待线程的阻塞状态。也就是说 wait相当于调用 await(), notifyAll 等价于 condition.signalAll(), 上面的Demo改为下面这样
>
> ```jav
> class Test implements Runnable {
>     private Boolean on;
> 
>     public Test(Boolean on) {
>         this.on = on;
>     }
> 
>     @Override
>     public void run() {
>         new MyClass().getData(on);
>     }
> }
> 
> public class MyClass {
> 
>     public static void main(String[] args) {
>         Thread a = new Thread(new Test(true));
>         Thread b = new Thread(new Test(false));
>         a.start();
>         b.start();
>     }
> 
>    synchronized  void getData(Boolean on) {
>         System.out.println("我被锁住了");
>         try {
>             while (on) {
>                 System.out.println("我阻塞了,放弃锁");
>                 wait();
>             }
>              notifyAll();
>             System.out.println("解除阻塞");
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
>     }
> 
> 
> }
> ```



#### 同步代码块

> 每一个java对象都有一个锁，线程可以调用同步方法来获得锁。还有一种机制可以获得锁，那就是使用一个同步代码块。
>
> ```this
>  synchronized (this){
>         
>  }
> ```
>
> 同步代码块是非常脆弱的，通常不推荐使用。一般实现同步最好使用 java.util.concurrent包下提供的类，比如阻塞队列。如果同步方法适合你的程序，那么请尽量使用 同步方法，这样可以减少编写代码的数量，减少出错的概率。如果特别需要使用Lock/Condition结构提供的独有特性是，才使用Lock/Condition.

#### volatile

> 有时仅仅为了读写一个或两个实例域就使用同步的话，显得开销过大；而volatile 关键字为实例域的同步访问提供了免锁的机制。如果声明一个域 为 volatile ，那么编译器和虚拟机就知道该域是可能被另一个线程并发更新的。
>
> 再学习volatile之前，我们需要了解一下内存模型的相关概念以及并发编程中的3个特性：原子性，可见性，有序性
>
> 1. Java的内存模型
>
>    > Java中的堆内存用来存储对象实例，堆内存是被所有线程共享的运行时内存区域，因此，他存在内存可见性的问题。而局部变量，方法定义的参数则不会再线程之间共享，他们不会有内存可见性的问题，也不受内存模型的影响。
>    >
>    > Java内存模型定义了线程和主存之间的抽象关系：线程之间的共享变量存储在主存中，每一个线程都有一个私有的本地内存，本地内存中存储了该线程共享变量的副本。需要注意的是本地内存是Java 内存模型的一个抽象概念，其实并不真实存在，它涵盖了缓存，写缓冲区，寄存器等区域。Java内存模型控制线程之间的通信，他决定一个线程对主存共享变量的写入核实对另一个线程可见。
>    >
>    > ![1551599926824](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1551599926824.png)
>    >
>    > 线程A 和 线程B 之间若要通信的话，必须要经历下面两个步骤：
>    >
>    > 1. 线程A把线程A本地内存中更新过的共享内存刷新到主存中去。
>    >
>    > 2. 线程 B到主存中去读取线程A之前已更新过的共享变量。由此可见，如果我们执行下面的语句： i=3；
>    >
>    >    执行线程必须先在自己的工作线程中对变量 i 所在的缓存进行赋值操作，然后再写入主存中，而不是直接将数值3写入到主存中
>
> 2. 原子性，可见性和有序性
>
>    > 1. 原子性
>    >
>    >    > 对基本数据类型的变量的读取和赋值时原子性操作，即这些操作是不可以被中断的，要么执行完毕，要么就不执行。看一下下面的代码，如下：
>    >    >
>    >    > ```java
>    >    > x=3;		//语句1
>    >    > y=x;		//语句2
>    >    > x++;		//语句3
>    >    > ```
>    >    >
>    >    > 在上面3个语句中，只有语句1是原子性操作，其他两个语句都不是原子性操作。
>    >    >
>    >    > 语句2虽说很短，但他包含了两个操作，它先读取x的值，在将x的值写入工作内存。读取x的值以及将x的值写入工作内存这两个操作单拿出来都是原子性操作，但是合起来就不是原子性操作了。
>    >    >
>    >    > 语句3包含3个操作： 读取x的值，对x的值进行+1，向工作内存写入新值。
>    >    >
>    >    > **所以，当一个语句包含多个操作时，就不是原子性操作，只有简单的读取和赋值(将数字赋给某个变量)才是原子性操作。**
>    >
>    > 2. 可见性
>    >
>    >    > 可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果    ，另一个线程马上就可以看到。当一个共享变量被 volatie修饰是，它会保证修改的值立即被更新到主存，所以随其他线程是可见的。当有其他线程需要读取该值时，其他线程会去主存中读取新值。而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，并不会立即写入主存，何时被写入主存也是不确定的。当其他线程去读取该值时，此时主存中可能还是原来的旧值，这样就无法保证可见性。
>    >
>    > 3. 有序性
>    >
>    >    > Java内存模型允许编译器和处理器对指令进行重排序，虽然重排过程不会影响到单线程执行的正确性，但是会影响到多线程并发执行的正确性。这时可以通过 valatie 来保证有序性，除了 voliatie ，也可以通过 synchronized 和 Lock 来保证有序性。syncheonized 和 Lock 保证每个时刻只有一个线程执行同步代码，这相当于让线程顺序执行同步代码，从而保证了有序性。
>
> 3. Volatile 关键字
>
>    > 当一个共享变量被 volatile 修饰之后，其就具备了两个含义，一个是线程修改了变量的值时，变量的新值对其他线程是立即可见的。换句话说，就是不同线程对这个变量进行操作时具有可见性。另一个含义是禁止使用指令重排序。
>    >
>    >
>    >
>    > 假设线程1先执行，线程2后执行
>    >
>    > ```java
>    > 		//线程1
>    >         boolean stop=false;
>    >         while (!stop){
>    >             
>    >         }
>    >         
>    >         //线程2
>    >         stop=true;
>    > ```
>    >
>    > 上面这段代码可以用于中断线程，但是这段代码不一定会将线程中断。虽然概率很小。
>    >
>    > 为什么说有可能无法中断线程呢？
>    >
>    > **每个线程在运行时都有私有的工作内存，因此线程1在运行时会将stop变量的值复制一份放在私有的工作内存中。当线程2更改了Stop变量的值后，线程2突然需要去做其他的操作，这时就无法将更改的Stop变量写入到主存中，这样线程1就不会知道线程2对Stop变量进行了更改，因此线程1就会一直循环下去。当Stop用volatile修饰之后，那么情况就变的不同了，当线程2进行修改时，会强制将修改的值立即写入主存，并且会导致线程1的工作内存中变量stop的缓存无效，这样线程1再次读取stop的值时就会去主存读取。**
>    >
>    >
>    >
>    > **volatile**不保证原子性
>    >
>    > 我们知道 volatile 保证了操作的可见性，但是是否能保证原子性呢？
>    >
>    > ```java
>    > class A{
>    >     public volatile int a=0;
>    >     public   void setA(){
>    >         a++;
>    >     }
>    > }
>    > 
>    > public class  MyClass{
>    >     public static void main(String[] args) {
>    >         final A a=new A();
>    >         for (int i=0;i<10;i++){
>    >             new Thread(new Runnable() {
>    >                 @Override
>    >                 public void run() {
>    >                     for (int i=0;i<1000;i++){
>    >                         a.setA();
>    >                     }
>    >                 }
>    >             }).start();
>    >         }
>    >         //如果有子线程就让出资源，保证所有子线程都执行完
>    >         while (Thread.activeCount()>2){
>    >             Thread.yield();
>    >         }
>    >         System.out.println(a.a);
>    >     }
>    > }
>    > ```
>    >
>    > 这段代码每次运行，结果都不一致。因为自增不具备原子性，它包括读取变量的原始值，进行+1，写入工作内存。也就是说，自增操作的3个子操作可能会分隔开执行。假如某个时刻变量inc的值为9，线程1对变量进行自增操作，线程1先读取了变量inc的原始值，然后线程1被阻塞了。之后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，然后进行加1操作，并把10写入工作内存，最后写入主存。随后线程1接着进行+1操作，因为线程1在此前已经读取了 inc 的值为9，所以不会再去主存读取最新的数值，线程1对 inc 进行+1操作后 inc 的值为10，然后将10 写入工作内存，最后写入主存。两个线程分别对 inc 进行了一次自增操作后，inc 的值只增加了1，因此自增操作不是原子性操作， volatile也无法保证对变量的操作是原子性的。
>    >
>    >
>    >
>    > **volatile保证有序性**
>    >
>    > volatile关键字能禁止指令重排序，因此 volatile 能保证有序性。volatile 关键字禁止指令重排序有两个含义：**一个是当程序执行 volatile 变量的操作时，在其前面的操作已经全部执行完毕，并且结果会对后面的操作可见，在其后面的操作还没有进行，在进行指令优化时，在 volatile 变量之前的语句不能在 volatile 变量后面执行；同样，在volatile 变量之后的语句也不能在 volatile变量前面执行**。
>
> 4. 正确使用volatile 关键字
>
>    > **synchronized关键字可防止多个线程同时执行一段代码，那么这就会很影响程序执行效率。而 volatile 关键字在某些情况下的性能要优于 synchronized 。但是要注意 volatile 关键字是无法替代 synchronized 关键字的，因为 volatile 关键字无法保证操作的原子性。**通常来说，使用 volatile 必须具备以下两个条件：
>    >
>    > 1. 对变量的写操作不会依赖于当前值
>    > 2. 对变量没有包含在具有其他变量的不变式中。
>    >
>    > 第一个条件就是不能是自增，自减等操作，因为 volatile不保证原子性。
>
>    ##### volatile使用场景
>
>    > 1. 状态标识
>    >
>    >    ```java
>    >     volatile  boolean on;
>    >        public void setOn(){
>    >            on=true;
>    >        }
>    >        public void Print(){
>    >            while(!on){
>    >                System.out.println("123");
>    >            }
>    >        }
>    >    ```
>    >
>    >    使用标识符,终止线程
>    >
>    > 2. 双重检查模式 （DCL）
>    >
>    >    ```java
>    >    class Singleton{
>    >        private volatile static Singleton instance = null;
>    >     
>    >        private Singleton() {
>    >     
>    >        }
>    >     
>    >        public static Singleton getInstance() {
>    >            if(instance==null) {
>    >                synchronized (Singleton.class) {
>    >                    if(instance==null)
>    >                        instance = new Singleton();
>    >                }
>    >            }
>    >            return instance;
>    >        }
>    >    }
>    >    ```
>    >
>    >    在上面的单例模式中，为什么要用valoatile 修饰？
>    >
>    >    因为 instance=new Singleton()，并非是一个原子操作，事实上在 JVM中这句话大概做了3件事
>    >
>    >    1. 给 instance 分配内存
>    >    2. 调用 Singletion 的构造函数来初始化成员变量
>    >    3. 将 instance 对象指向分配的内存空间。(执行完这步 instance 就为 非null 了)
>    >
>    >    但是JVM 的即时编译器中存在指令重排序的优化，也就是说上面的第二步和第三步顺序是不确定的，一旦2,3，顺序乱了，这时候有另一个线程调用了方法，结果虽然非null,但是未初始化，所以会直接报错。
>

### 

### 阻塞队列

> 阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也从容器里拿元素。

1. ##### 常见阻塞场景

   > 1. 当队列中没有数据的情况下，消费者端的所有线程都会被自动堵塞(挂起),直到有数据放入队列。
   >
   > 2. 当队列中填满数据的情况下，生产者端的所有线程都会被自动阻塞(挂起)，直到队列中有空的位置，线程被自动唤醒。
   >
   >    支持以上两种阻塞场景的被称为阻塞队列。

2. BlockingQueue 方法

   > 放入数据：
   >
   > - off (anObject) :表示如果可能的话，将addObject 加到 BlockingQueue里。即如果BlockingQueue 可以容纳，则返回 true，否则 返回false .(本方法不阻塞当前执行方法的线程)
   > - offer(E o,long timeout,TimeUnit unit):可以设定等待时间。如果在指定的时间内还不能忘队列中加入 BlockingQueue ，则返回失败。
   > - put(anObject): 把anObject 加到BlockQueue 里，如果 BlockQueue 没有空间，则调用此方法的线程被阻断直到 BlockingQueue 里面有空间再继续。
   >
   > 获取数据：
   >
   > 1. poll(time) :取走 BlockingQueue 里排在首位的对象，若不能立即去除，则可以等 time 参数规定的时间，取不到是 返回 null.
   > 2. poll(long timeout,TimeUnit unit): 从BlockingQueue 取出一个队首的对象，如果在指定时间内，则立即返回队列中的数据。否则直到时间超时还没有数据可取，返回失败。
   > 3. take() ： 取走BlockingQueue 里排在首位的对象。若 BlockingQueueue为空，则阻断进入等待状态，直道BlockingQueue 有新的数据被加入。
   > 4. drainTo(): 一次性从 BlockingQueue 获取所有可用的数据对象(还可以指定数据的个数).通过该方法，可以提升获取数据的效率；无序多次分批加速或释放锁。

#### Java中的阻塞队列

在java中提供了7个阻塞队列，分别如下

1. ArrayBlockingQueue : 由数组结构组成的有界阻塞队列
2. LinkedBlockingQueue: 由链表结构组成的有界阻塞队列
3. PriorityBlockingQueue:  支持优先级排序的误解阻塞队列
4. DelyQueue :  使用优先级队列实现的无界阻塞队列
5. SynchronousQueue :  不存储元素的阻塞队列
6. LinkedTransferQueue:  由链表结构组成的无界阻塞队列
7. LinkedBlockingQueue:  由链表结构组成的双向阻塞队列。

详细介绍

1. ArryBlockingQueue

   > 它是用数组实现的有界阻塞队列，并按照先进先出(FIFO) 的原则对元素进行排序。默认情况下不保证线程公平的访问队列。 **公平访问队列就是指阻塞的所有生产者线程或消费线程，当队列可用是，可以按照阻塞的先后顺序访问队列。即先阻塞的生产者线程，可以先往队列里插入元素；先阻塞的消费者线程，可以先从队列里获取元素** 。通常情况下为了保证公平性会降低通吐量。我们可以使用以下代码创建一个公平的阻塞队列。
   >
   > ```java
   >  ArrayBlockingQueue fairQueue=new ArrayBlockingQueue(2000,true);
   > ```

2. LinkedBlockingQueue

   > 它是基于链表的阻塞队列，同ArrayListBlockingQueue类似，此队列按照先进先出(FIFO)的原则对元素进行排序，其内部也维持着一个数据缓冲队列(该队列由一个链表构成). 当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓冲在队列内部，而生产者立即返回；只有当队列缓冲区达到缓存容量的最大值是(LinkedBlockingQueue可以通过构造方法指定该值)，才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒。反之对于消费者这端的处理也基于同样的原理。
   >
   > 而LinkedBlockingQueue 之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步。 这也意味着在高并发的情况下生产者和消费者可以并行的操作队列中的数据，以此来提高整个队列的并发性能。
   >
   > 我们需要注意的是，如果构造一个 LinkedBlockingQueue 对象，而没有指定其容量大小，LinkedBlockingQueue 会默认一个类似无限大小的容量(Integer.MAX_VALUE).这样一来，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。ArrayBlockingQueue 和 LinkedBlockingQueue 是两个最普通也是最常用的阻塞队列。一般情况下，在处理多线程的 生产者-消费者问题是，使用这两个类足以。
   >
   > **Demo**
   >
   > ```java
   > //生产者
   > 
   > public class Shengchan implements Runnable{
   > 
   >     private volatile  boolean isRunnage =true;      //是否在运行状态
   >     private BlockingQueue<String> queue;        //阻塞队列
   >     //原子方式更新
   >     private static AtomicInteger cout=new AtomicInteger();      //自动更新的值
   >     private static final int De=1000;
   >     @Override
   >     public void run() {
   >         String data=null;
   >         Random r=new Random();
   >         System.out.println("启动生产者线程");
   >         while (isRunnage){
   >             System.out.println("正在生产");
   >             try {
   >                 Thread.sleep(r.nextInt(De));        //取0-1000的一个随机数
   >                 data="data"+cout.incrementAndGet();     //以原子方式将 cout+1
   >                 System.out.println("将数据："+data+"放入队列...");
   >                 if (!queue.offer(data,2, TimeUnit.SECONDS)){        //设定的等待时间，如果超过2S还没加进去返回true
   >                     System.out.println("放入数据失败："+data);
   >                 }
   >             } catch (InterruptedException e) {
   >                 e.printStackTrace();
   >                 Thread.currentThread().interrupt();
   >             }finally {
   >                 System.out.println("退出生产者线程");
   >             }
   >         }
   >     }
   >     public void stop(){
   >         isRunnage=false;
   >     }
   > 
   >     public Shengchan(BlockingQueue<String> queue) {
   >         this.queue = queue;
   >     }
   > }
   > 
   > ```
   >
   > ```java
   > //消费者
   > public class Xiaofeizhe implements Runnable{
   >     private BlockingQueue<String> queue;
   >     private static final int DEFAULT=1000;
   > 
   >     //构造函数
   >     public Xiaofeizhe(BlockingQueue<String> queue) {
   >         this.queue = queue;
   >     }
   > 
   > 
   >     @Override
   >     public void run() {
   >         System.out.println("启动消费者线程");
   >         Random r=new Random();
   >         boolean isRunnag=true;
   >         while(isRunnag){
   >             System.out.println("正在从队列获取数据");
   >             try {
   >                 String data=queue.poll(2, TimeUnit.SECONDS);        //有数据是直接从队列的对首取走，无数据是阻塞，在2s内有数据，取走，超过2s还没数据，返回失败
   >                 if (data != null) {
   >                     System.out.println("拿到数据"+data);
   >                     System.out.println("正在消费数据"+data);
   >                     Thread.sleep(r.nextInt(DEFAULT));
   >                 }else {
   >                     //超过2s还没数据，认为所有生产线程都已经退出，自动退出消费线程。
   >                     isRunnag=false;
   >                 }
   >             } catch (InterruptedException e) {
   >                 e.printStackTrace();
   >                 //中断线程
   >                 Thread.currentThread().interrupt();
   >             }finally {
   >                 System.out.println("退出消费者线程");
   >             }
   >         }
   >     }
   > 
   > }
   > 
   > ```
   >
   > ```java
   > //测试类
   > public class BlockingQueueTest {
   >     public static void main(String[] args) throws InterruptedException {
   >         //声明一个容量为10的缓存队列
   >         BlockingQueue<String> queue=new LinkedBlockingDeque<>(10);
   > 
   >         //new了3个生产者和一个消费者
   >         Shengchan s1=new Shengchan(queue);
   >         Shengchan s2=new Shengchan(queue);
   >         Shengchan s3=new Shengchan(queue);
   >         Xiaofeizhe x1=new Xiaofeizhe(queue);
   > 
   >         //借助Excutors
   >         //创建线程池
   >         ExecutorService service= Executors.newCachedThreadPool();
   > 
   >         service.execute(s1);
   >         service.execute(s2);
   >         service.execute(s3);
   >         service.execute(x1);
   > 
   >         //执行10s
   >         Thread.sleep(10*1000);
   >         s1.stop();
   >         s2.stop();
   >         s3.stop();
   > 
   >         Thread.sleep(2000);
   >         //退出Excutor
   >         service.shutdown();
   >     }
   > }
   > ```

3. PriorityBlockingQueue

   > 它是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。这里可以自定义实现 comepareTo() 方法来指定元素进行排序规则；或者初始化PriorityBlockingQueue 时，指定构造参数 Comparator 来对元素进行排序。但其不能保证痛优先级元素的顺序。

4. DelayQueue

   > 它是一个支持延时获取元素的无界阻塞队列。队列使用 PriorityQueue 来实现。队列中的元素必须实现Delayed接口。创建元素时，可以指定元素到期的时间，只有在元素到期时才能从队列中取走。

5. SynchronousQueue

   > 它是一个不存储元素的阻塞队列。每个插入操作必须等待另一个线程的移除操作，同样任何一个移除操作都等待另一个线程的插入操作。因此队列内部其实没有任何一个元素，或者说容量是0，严格来说它并不是一种容器。由于队列没有容量，因此不能调用peek操作(返回队列的头元素)。

6. LinkedTransferQueue

   > 它是一个由链表结构组成的无界阻塞TransferQueue队列。LinkedTransferQueue实现了一个重要的接口TransferQueue。该几口含有5个方法，其中有3个重要方法，分别如下。
   >
   > 1. transfer(E e) :若当前存在一个正在等待获取的消费者线程，则立刻将元素传递给消费者；如果没有消费者在等待接收数据，就会将元素插入到队列尾部，并且等待进入阻塞状态，知道有消费者线程取走该数据。
   > 2. tryTransfer(E e): 若当前存在一个正在等待获取的消费者线程，则立刻将元素传递给消费者；若不存在，则返回 false ，并且不进入队列，这是一个不阻塞的操作。与 transfer 方法不同的是，tryTransfer方法无论消费者是否接受，其都会立即返回；而 transfer方法则是消费者接受了才返回。
   > 3. tryTransfer(E e,lomh timeout,TimeUnit unit): 若当前存在一个正在等待获取的消费者线程，则立即将元素传递给消费者；若不存在则将元素插入到队列尾部，并且等待消费者线程取走该元素。若在指定的超时时间内元素未被消费者线程获取，则返回 false ；若在指定的超时时间内其被消费者线程获取，则返回 true.

7. LinkBlockingDeque

   > 它是一个由链表结构组成的双向阻塞队列。双向队列可以从队列的两端插入和移除元素，因此在多线程同时入队时，也就减少了一半的竞争。由于是双向的，因此 LinkedBlockingDeque 多了addFrist,addLast,offerFitrst,offerLast,peekFirst,peekLast等方法。其中，以Frist单词结尾的方法，表示插入，获取或移除双端队列的第一个元素；以Last单词结尾的方法，表示插入，获取或移除双端队列的最后一个元素。



### 线程池

> 在编程中经常会使用线程池来异步处理任务，但是每个线程池的创建和销毁都有一定的开销。如果每次执行一个任务都需要开一个新线程去执行，则这些线程的创建和销毁将消耗大量的资源；并且线程都是各自为政，很难对其进行控制，更何况有一堆的线程在执行。这时就需要线程池来对线程进行管理。在java中提供了 Executor框架用于把任务的提交和执行解耦，任务的调教交给 Runnable 和 Callable,而Executor 框架用来处理任务。Executor 框架中最核心的成员就是 ThreadPoolExeeutor,它是线程池的核心实现类。

#### ThreadPoolExector

> 可以通过 ThreadPoolExector来创建一个线程池，ThreadPoolExecutor 类一共有4个构造方法，其中拥有最多参数的构造方法如下。
>
> ![1552211314605](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1552211314605.png)
>
> - corePoolSize: 核心线程数。默认情况下线程池是空的，只有任务提交时才会创建线程。如果当前运行的线程数少于 corePoolSize ，则创建新线程来处理任务；如果等于或者多于 crePoolSize ，则不再创建。如果调用线程池的 prrestartAllcoreThread 方法，线程池会提前创建并启动所有核心线程来等待任务。
> - maximumPoolSize: 线程池允许创建的最大线程数。如果任务队列满了并且线程数小于 maximumPoolSize时，则线程池仍旧会创建新的线程来处理任务。
> - keepAliveTime: 非核心线程闲置的超过时间。超过这个时间则回收。如果任务很多，并且每个任务的执行事件很短，则可以调大KeepAliveTime 来提高线程的利用率。另外，如果设置allowCoreThreadTimOut属性为 true 时，keepAliveTime 也会应用到核心线程上。
> - TimeOut: keepAliveTime 参数的时间单位。可选的单位有 天(days),小时(hours)，分钟（minutes），秒（seconds）,毫秒(milliseconds)等。
> - workQueue： 任务队列。如果当前线程数大于corePoolSize，则将任务添加到此任务队列中。该任务队列是BlockingQueue 类型，也就是阻塞队列。
> - ThreadFactory: 线程工厂。 可以用线程工厂给每个创建出来的线程设置名字。一般情况下无序设置改参数。
> - RejectdExecutionHandler: 饱和策略。这是当任务队列和线程池都满了时所采用的应对策略，默认是AbordPolicy,表示无法处理新任务，并抛出 RejectedExecutionException异常。此处还有3种策略，他们分别如下。
>   1. CallerRunsolicy ： 用调用者所在的线程来处理任务。此策略提供加单的反馈控制机制，能够缓解新任务的提交速度。
>   2. DiscardPolicy: 不能执行的任务，并将该任务删除
>   3. DiscardOldestPolicy:  丢弃队列最近的任务，并执行当前的任务。

#### 线程池的处理流程和原理

> 当向线程提交任务时，线程池是如何处理的呢？
>
> ![1552213547081](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1552213547081.png)
>
> 1. 提交任务后，线程池先判断线程数时候达到了核心线程数。如果未达到核心线程数，则创建核心线程处理任务；否则，就执行下一步操作。
> 2. 接着线程池判断任务队列是否满了。如果没满，则将任务添加到队列中；否则，就执行下一步操作。
> 3. 接着因为线程池判断满了，线程池就判断线程数是否达到了最大线程数。如果未达到，则创建非核心线程处理任务；否则，就执行饱和策略，默认会抛出RejectedExecutionException异常。
>
> ![1552213917602](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1552213917602.png)
>
> 1. 如果线程池中的线程数未达到核心线程数，则创建核心线程处理任务。
> 2. 如果线程数大于或者等于核心线程数，则将任务加入任务队列，线程池中的空闲线程会不断地从任务队列中取出任务进行处理。
> 3. 如果任务队列满了，并且线程数没有达到最大线程数，则创建非核心线程去处理。
> 4. 如果线程数超过了最大线程数，则执行饱和策略。

#### 线程池的种类

通过直接或者间接的配置 ThreadPoolExecutor 的参数可以创建不同类型的ThreadPoolExecutor,其中有4种线程池比较常用，分别是

**FixedThreadPool**

> FixedThreadPool是可重用固定线程数的线程。在Executors 类中提供了创建FixedThreadPool 的方法，如下所示，
>
> ```
>     public static ExecutorService newFixedThreadPool(int var0) {
>         return new ThreadPoolExecutor(var0, var0, 0L, TimeUnit.MILLISECONDS, new 			LinkedBlockingQueue());
>     }
> ```
>
> FixedThreadPool 的 corePoolSize 和maximumPoolsize 都设置为 创建 FixedThreadPool 指定的参数Threads ,也就意味着FixedThreadPool 只有核心线程，并且数量是固定的，没有非核心线程。keepAliveTime 设置为0L,意味着多余的线程会被立即终止。因为不会产生多余的线程，所以keepAliveTime 是无效的参数。另外，任务队列采用了无界阻塞队列 LinkBlockingQueue（容量默认为Integer.MAX_VALUE）。FixedThreadPool的execute 方法执行
>
> ![1552223643448](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1552223643448.png)
>
> **当执行execute方法时，如果当前运行的线程未达到 corePoolSize（核心线程数）时就创建核心线程来处理任务，如果达到了核心线程数则将任务添加·到·LinkedBlockingQueue中，FixedThreadPool 就是一个有固定核心线程的线程池，并且这些线程不会被回收。当线程数超过 corePoolize 时，就将任务存储在任务队列中，当线程池有空闲线程是，则从任务队列中去任务执行。 **



**CachedThreadPool**

> CachedThreadPool 是一个根据需要创建的线程池。
>
> ```
>   public static ExecutorService newCachedThreadPool() {
>         return new ThreadPoolExecutor(0, 2147483647, 60L, TimeUnit.SECONDS, new 		SynchronousQueue());
>     }
> ```
>
> CachedThreadPool 的 corePoolSize 为0,maximumPoolSize 设置为 Integer.Max_value.这意味着CachedThreadPool 没有核心线程，非核心线程是无界的。 keepAliveTime 设置为 60L，则空闲线程等待任务的最长时间为60s 。在此用了阻塞队列 SynchronousQueue ，它是一个不存储元素的阻塞队列，每个插入操作必须等待另一个线程的移除操作，同样任何一个移除操作都等待另一个线程的插入操作。
>
> ![1552226148891](C:\Users\15094\AppData\Roaming\Typora\typora-user-images\1552226148891.png)
>
> 当执行 excute方法时，首先会执行 SynchrnousQueue的 offer 方法来提交任务，并且查询线程池找那个是否有空线程执行 SynuchronousQueue 的 poll 方法唉移除任务。如果有子配对成功，将任务交给这个空闲的线程处理；如果没有则配对失败，创建新的线程去处理任务。当线程池中的线程空闲时，它会执行 SynchronousQueue 的poll 方法，等待 SynchrnusQueue 中新提交的任务。如果超过60s没有新任务提交到 SynchrnusQueue ，则这个空闲线程将终止。因为 maximumPoolSize 是吴洁德，所以如果提交的任务大于线程池中线程处理任务的速度就会不断创建新线程。另外，每次提交任务都会立即有线程去处理。所以 CachedThreadPool 比较适合于大量需要立即执行并且耗时较少的任务。

