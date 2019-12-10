# Java动态代理其实很简单

在使用Java 动态代理时，一直很迷惑，什么是动态代理，动态在了那里？它和静态代理的区别是什么？但是很遗憾，没有找到一个能真正简单明了的告诉我原因的博客，于是决定自己动手，分析一下。

首先，本篇的主要围绕点如下：

![image-20191210165656970](https://tva1.sinaimg.cn/large/006tNbRwly1g9rqbzq98qj316u0hwmyn.jpg)

当然，对于其中的具体实现，并不会太去关注，本篇博客主旨是简单通俗的告诉你，什么是动态代理，它的流程是什么。



#### 首先，什么是动态代理？

我复制一个大佬的解释如下：

利用Java的反射技术(Java Reflection)，在运行时创建一个实现某些给定接口的新类（也称“动态代理类”）及其实例（对象）,代理的是接口(Interfaces)，不是类(Class)，也不是抽象类。在运行时才知道具体的实现，spring aop就是此原理。

这个解释有问题吗，没有问题，很简单明了，但是对于新同学，特别是刚入行没多久的，带来的感受只有一个，那就是MMP？

拒绝官方话语，让技术变得简单通俗，这是我们每个技术人都追寻的目标吧。



好了开始我们的教程：



我们用一个简单的demo开始。

**首先，这是一个接口：**

```java
public interface Subject {
    void play();
}
```

很简答，它的作用我们只是为了打印一句话。



**接着有一个扩展类，实现了它：**

```java
public class XiaoQing implements Subject {
    @Override
    public void play() {
        System.out.println("我是小强，我要去跑步");
    }
}
```



**最后，看我们具体的处理类**

```java
public class TestProxy implements InvocationHandler {
    private Subject subject;

    public TestProxy(Subject subject) {
        this.subject = subject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("我是动态代理");
        method.invoke(subject, args);
        System.out.println("结束");
        return null;
    }

  
  
    public static void main(String[] args) {
        XiaoQing xiaoQing = new XiaoQing();
        TestProxy testProxy = new TestProxy(xiaoQing);
        //拿到代理对象
        Subject subject = (Subject) Proxy.newProxyInstance(xiaoQing.getClass().getClassLoader(), xiaoQing.getClass().getInterfaces(), testProxy);
        subject.play();
    }
}

```

先放下main方法不看，我们关注 TestProxy 这个类，它实现了一个 InvocationHandler 接口，并实现了 invoke 方法。



**我们看一下 invoke方法：**

熟悉反射的同学应该会发现，这里我们调用了 method.invoke()方法，并传入了我们的接口对象和参数args.从而调用我们的接口中的方法。也就是说从这里我们可以插入相应的执行前后方法。



#### 继续我们的流程：

**我们看一下main方法，也就是我们的测试入口：**

里面也很简单，我们主要关注 **Proxy** 这个类。

```java
public class Proxy implements java.io.Serializable {
		。。。
    protected InvocationHandler h;

   。。。
```

可以发现，它内部持有一个 InInvocationHandler 接口对象，那这个对象是干啥的呢，我们点进去看一下：



很简单，里面只有一个方法，invoke,也就是俗称的**拦截方法**。

```java
interface InvocationHandler{
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

```
 InvocationHandler 是由代理实例的<i>调用处理程序</ i>实现的接口。 * * <p>每个代理实例都有一个关联的调用处理程序。 在代理实例上调用某个方法时，该方法的调用将被编码并调度到其调用处理程序的{@code invoke} *方法
```

**简单粗暴，也就是说我们每个代理类都有一个关联的调用处理类，而这个接口的作用就是将我们调用的代理方法调度到真实调用处去。也就是调用它的 invoke 方法。**



可能听得有点绕，没关系，我们先暂且持有这个疑问，去看一下 Proxy.newProxyInstance方法:

```java
   public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
```

 **通过注释我们明白，它的作用是用来返回我们的代理类实例。注意我们传进去的三个参数与，第一个，classLoader，类加载器，第二个 就是我们真实对象实现的接口类，第三个就是我们自己实现的处理类。**



有了上面的解析，大家可能还会存在疑惑，那下面我们去看一下动态代理生成的代理类，一探究竟：

这里碍于原因，我们从网上找一段生成的代理类，来看一下，差别也不会很大：

```java
。。。
public final class $Proxy0
extends Proxy
implements HelloService {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler invocationHandler) {
        super(invocationHandler);
    }

    public final boolean equals(Object object) {
        try {
            return (Boolean)this.h.invoke((Object)this, m1, new Object[]{object});
        }
        catch (Error | RuntimeException v0) {
            throw v0;
        }
        catch (Throwable var2_2) {
            throw new UndeclaredThrowableException(var2_2);
        }
    }

    public final String sayHello(String string) {
        try {
            return (String)this.h.invoke((Object)this, m3, new Object[]{string});
        }
        catch (Error | RuntimeException v0) {
            throw v0;
        }
        catch (Throwable var2_2) {
            throw new UndeclaredThrowableException(var2_2);
        }
    }

   。。。
    }
}
```

我们看一下sayHello 方法，我们先将 sayHello 当做我们刚才接口中的 play方法，再观察内部实现：

其中 h 就是invocationHandler，当我们在调用接口play方法时，其实代理类，通过 invocationHandler对象 调用 invoke 方法并传入代理对象，方法，及我们调用时传递过来的参数，然后 定位到我们真实要调用的方法，从而我们可以在调用方法的前后就可以插入我们相应的代码实现动态代理。





### 最后，我们总结一下我们最开始时思维导图中的几个问题：

#### InvocationHandler ？为什么要实现它？

InvocationHandler 就相当于一个委托处理，我们实现它的 invoke 方法，然后便于我们可以动态的在 invoke 中插入自己的代码，实现相应逻辑。

#### Proxy.newProxyInstance？

Proxy.newProxyInstance 的主要作用就是通过反射生成我们的代理对象，从而将我们的代理对象关联到传入的 InvocationHandler对象上。

#### Invoke？

便于我们动态的插入自己的代码，并为了便于 我们生成的代理对象调用接口方法时，最后实则调用 invoke方法,并传入代理对象，真实方法和对应的参数。

***所以所谓的动态代理，实则是通过类加载器在初始化时动态生成了一个代理对象，这个代理对象实现了我们接口的所有方法，并在后续我们调用时，内部通过 invocationHandler 对象调用invoke方法从而定位到我们真实的调用位置，从而让我们可以在调用前后插入我们相应的代码。***

