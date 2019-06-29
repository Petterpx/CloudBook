# Java基础类库

#### 如何判断两个对象一定是同一个对象？

identityHashCode(Object x)方法，该方法返回指定对象的精确hashCode 值，也就是根据该对象的地址计算得到的 hashCode值。当某个类的 hashCode() 方法被重写后，该类实力的 hashCode() 就不能唯一的表示该对象，但通过 indentityHashCode() 方法返回的 hashCode 值，依然是 根据该对象的地址计算得到 hashCode 值。

```java
public class OK {
    public static void main(String[] args) {
        String s1 = new String("Petterp");
        String s2 = new String("Petterp");
        //String 重写了 HashCode() 方法，改为根据字符序列计算 hashcode 值。
        //因为s1 和 s2 是不同的字符串对象，所以他们的 hashcode() 方法返回值相同
        System.out.println(s1.hashCode()+"______"+s2.hashCode());
        System.out.println(System.identityHashCode(s1)+"_______"+System.identityHashCode(s2));
        String s3="Java";
        String s4="Java";
        //s3 和 s4 是相同的字符串对象，所以它们的 identyHashCode 值相同。
        System.out.println(System.identityHashCode(s3)+"_____"+System.identityHashCode(s4));
    }
}
```