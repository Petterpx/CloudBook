# finally与return 的执行顺序

Java 中异常捕获机制 try..catch..finally语句是不是一定会被执行？

**至少有两种情况下 finally语句是不会被执行的：**

- **try 语句没有被执行到，如在 try语句之前就返回了，这样 finally 语句就不会执行，这也说明了 finally语句被执行的必要而非充分条件是：相应的 try语句一定被执行到。**
- **在try块中有 System.exit(0);这样的语句，System.exit(0) 用来终止Java 虚拟机JVM,链JVM 都停止了，当然 finally 语句也不会被执行到。**

以Demo为例

```java
public class Test4 {
    static int mode;

    public static void main(String[] args) {
        Demo();
        System.out.println("mode-----"+mode);
    }
    private static int Demo() {
        try {
            System.out.println("我是第1个return，返回1");
            return mode = 1;
        } catch (Exception ignored) {

        } finally {
            System.out.println("我是finally");
        }
        System.out.println("我是第二个return，返回2");
        return mode = 2;
    }
}
```

执行结果

```java
我是第1个return，返回1
我是finally
mode-----1
```

从而可以证明，如果try语句被执行到了，如果try语句里有return语句，finally也会被执行，然后等finally执行完后再返回。

接着我们再修改，如果我们给 finally 语句中添加返回呢？

```java
public class Test4 {
    static int mode;

    public static void main(String[] args) {
        Demo();
        System.out.println("mode-----"+mode);
    }
    private static int Demo() {
        try {
            System.out.println("我是第1个return，返回1");
            return mode = 1;
        } catch (Exception ignored) {

        } finally {
            System.out.println("finally语句中--------mode="+mode);
            System.out.println("我是finally,返回3");
            return mode=3;
        }
//        System.out.println("我是第二个return，返回2");
//        return mode = 2;
    }
}
```

执行结果

```java
我是第1个return，返回1
finally语句中--------mode=1
我是finally,返回3
mode-----3
```

如果finally语句中添加了return,那么就按照正常的执行逻辑走，不过这个时候try语句后的return就失效了，编译器报错，因为不会被执行到，变成不可达语句了，