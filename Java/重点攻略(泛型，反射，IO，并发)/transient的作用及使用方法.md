# transient的作用及使用方法

> 我们都知道一个对象只要实现了 Serilizable接口，这个对象就可以被序列化，java 的这种序列化为开发者提供了很多便利，我们可以不必关心具体序列化的过程，只要这个类实现了 Serilizable 接口，这个类的所有属性和方法都会自动序列化。
>
> 然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息，为了安全起见，不希望在网络上操作（主要涉及到序列化操作，本地序列号缓冲也适用）中被传输，这些信息对应的变量就可以加上 transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。
>
> 总之，java 的transient关键字为我们提供了遍历，你只需要实现 Serilizable 接口，将不需要序列化的属性前添加关键字 transient,序列化对象的时候，这个属性就不会序列化到指定的目的地中。

```java
public class Test4 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user=new User();
        user.setUsername("Petterp");
        user.setPasswd("123456");
        System.out.println("序列化");
        System.out.println("username "+user.getUsername());
        System.out.println("password "+user.getPasswd());

        ObjectOutputStream os=new ObjectOutputStream(new FileOutputStream("d:a.txt"));
        os.writeObject(user);       //将user 对象写进文件
        os.flush();
        os.close();

        ObjectInputStream is=new ObjectInputStream(new FileInputStream("d:a.txt"));
        user= (User) is.readObject();       //从流中读取 User的数据
        is.close();

        System.out.println("反序列化后");
        System.out.println("username "+user.getUsername());
        System.out.println("password "+user.getPasswd());
    }
}
class User implements Serializable{
    private static final long serialVersionUID = -2198690960463122880L;
    private String username;
    private transient String passwd;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPasswd() {
        return passwd;
    }

    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }
}
```



## transient使用小结

- 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

- transient 关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被 transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现 Serializable 接口。

- 被 transient 关键字修饰的变量不再能被序列化，一个**静态变量**不管是否被 transient 修饰，均不能被序列化。

  第三点可能有些人很迷惑，因为发现在 User类中的 username 字段前加上 static 关键字后，程序运行结果依然不变，即 static 类型的 username 也读出来了原来的值，这不与我们说的第三点矛盾吗？实际上是这样的：第三点确实没错(一个静态变量不管是否被 transient修饰，均不能被序列化)，反序列化后类中 static 型变量 username 的值为当前 JVM 中对应  static 变量的值，**这个值是 JVM 中的不是反序列化得出的。**

  ```java
  public class Test4 {
      public static void main(String[] args) throws IOException, ClassNotFoundException {
          User user=new User();
          user.setUsername("Petterp");
          user.setPasswd("123456");
          System.out.println("序列化");
          System.out.println("username "+user.getUsername());
          System.out.println("password "+user.getPasswd());
  
          ObjectOutputStream os=new ObjectOutputStream(new FileOutputStream("d:a.txt"));
          os.writeObject(user);       //将user 对象写进文件
          os.flush();
          os.close();
  
          User.username="123";
          ObjectInputStream is=new ObjectInputStream(new FileInputStream("d:a.txt"));
          user= (User) is.readObject();       //从流中读取 User的数据
          is.close();
  
          System.out.println("反序列化后");
          System.out.println("username "+user.getUsername());
          System.out.println("password "+user.getPasswd());
      }
  }
  class User implements Serializable{
      private static final long serialVersionUID = -2198690960463122880L;
      public static String username;
      private transient String passwd;
  
      public String getUsername() {
          return username;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      public String getPasswd() {
          return passwd;
      }
  
      public void setPasswd(String passwd) {
          this.passwd = passwd;
      }
  }
  ```

  结果

  ```java
  username Petterp
  password 123456
  反序列化后
  username 123
  password null
  ```



### transient使用细节——被 transient关键字修饰的变量真的不能被序列化吗？

```java
public class Test4 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ExternalizableTest test=new ExternalizableTest();
        ObjectOutput out=new ObjectOutputStream(new FileOutputStream(new File("test")));
        out.writeObject(test);

        ObjectInput in=new ObjectInputStream(new FileInputStream(new File("test")));
        test= (ExternalizableTest) in.readObject();
        System.out.println(test.content);
    }
}
class ExternalizableTest implements Externalizable{
    private static final long serialVersionUID = 5661323170598296402L;
    public transient String content="我被序列化，不管是否被修饰";

    public ExternalizableTest() {
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {

    }

    @Override
    public void readExternal(ObjectInput in) throws ClassNotFoundException, IOException {

    }
}
```

结果

```java
我被序列化，不管是否被修饰
```

不是说类的变量被 transient关键字修饰以后将不能序列化了吗？

> 我们知道在 Java中，对象的序列化   可以通过两种接口来实现，若实现的是 Serializable 接口，则所有的序列化将会自动进行，若实现的是 Externalizable 接口，则没有任何东西可以自动序列化，需要在 writeExternalf  方法中进行手工指定所要序列化的变量，这与是否被 transient 修饰无关。因此第二个例子输出的是变量 conetent 初始化的内容，而不是 null;