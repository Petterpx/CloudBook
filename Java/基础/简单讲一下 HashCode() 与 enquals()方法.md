# 简单讲一下 HashCode() 与 equals()方法.

### 简单讲一下 hashCode 方法？

> - hashCode 的存在主要用于查找的快捷性，如 **Hashtable**, **HashMap** 等，hashCode 是用来在三列存储结构中确定对象的存储地址的。
> - 如果两个对象相同，就是适用于 euqals(java.lang.Object) 方法，那么这两个对象的 hashCode一定相同。
> - 如果对象的euqals 方法被重写，那么对象的 hashCode 也尽量重写，并且产生 hashCode 使用的对象，一定要和 equals 方法中使用的一致，否则就会违反上面提到的第二点。
> - 两个对象的 hashCode 相同，并不一定表示这两个对象就相同，也就是不一定适用于equals() 方法，只能够说明这两个对象在三列存储结构中，如 Hashtable.,他们存在同一个篮子里。
>
> ```java
> 1.hashcode是用来查找的，如果你学过数据结构就应该知道，在查找和排序这一章有  
> 例如内存中有这样的位置  
> 0  1  2  3  4  5  6  7    
> 而我有个类，这个类有个字段叫ID,我要把这个类存放在以上8个位置之一，
> 如果不用hashcode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。  
> 但如果用hashcode那就会使效率提高很多。  
> 我们这个类中有个字段叫ID,那么我们就定义我们的hashcode为ID％8，
> 然后把我们的类存放在取得得余数那个位置。比如我们的ID为9，
> 9除8的余数为1，那么我们就把该类存在1这个位置，
> 如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。
> 这样，以后在查找该类时就可以通过ID除 8求余数直接找到存放的位置了。  
>   
> 2.但是如果两个类有相同的hashcode怎么办那（我们假设上面的类的ID不是唯一的），
> 例如9除以8和17除以8的余数都是1，那么这是不是合法的，
> 回答是：可以这样。那么如何判断呢？在这个时候就需要定义 equals了。  
> 也就是说，我们先通过 hashcode来判断两个类是否存放某个桶里，
> 但这个桶里可能有很多类，那么我们就需要再通过 equals 来在这个桶里找到我们要的类。  
> 那么。重写了equals()，为什么还要重写hashCode()呢？  
> 想想，你要在一个桶里找东西，你必须先要找到这个桶啊，
> 你不通过重写hashcode()来找到桶，光重写equals()有什么用啊  
> ```
>
> ```java
> package java.lang;
> 
> public class Object {
> ·······
> 
>     /**
>      * 返回该对象的哈希码值。
>      * 支持此方法是为了提高哈希表（例如 java.util.Hashtable 提供的哈希表）的性能
>      * {@link java.util.HashMap}.
>      * <p>
>      * hashCode 的常规协定是:
>      * <ul>
>      * <li>在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，
>      *     必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。
>      *     从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
>      * <li>如果根据 equals(Object) 方法，两个对象是相等的，
>      *     那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果。
>      * <li>如果根据 equals(java.lang.Object) 方法，两个对象不相等，
>      *     那么对这两个对象中的任一对象上调用 hashCode 方法不 要求一定生成不同的整数结果。
>      *     但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。
>      * </ul>
>      * <p>
>      * 实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。
>      * （这一般是通过将该对象的内部地址转换成一个整数来实现的，
>      * 但是 JavaTM 编程语言不需要这种实现技巧。）
>      *
>      * @return  此对象的一个哈希码值。
>      * @see     java.lang.Object#equals(java.lang.Object)
>      * @see     java.lang.System#identityHashCode
>      */
>     public native int hashCode();
> ·······
> }
> ```
>
> 接下来看下面这个例子：
>
> ```java
> public class MyClass {
>  public static void main(String[] args) {
>      HashSet books=new HashSet();
>      books.add(new A());
>      books.add(new A());
>      books.add(new B());
>      books.add(new B());
>      books.add(new C());
>      books.add(new C());
>      System.out.println(books);
>  }
> }
> class A{
>  //类A的 equals 方法总是返回true,但没有重写其hashCode() 方法
>  @Override
>  public boolean equals(Object o) {
>      return true;
>  }
> }
> class B{
>  //类B 的hashCode() 方法总是返回1，但没有重写其equals()方法
>  @Override
>  public int hashCode() {
>      return 1;
>  }
> }
> class C{
>  public int hashCode(){
>      return 2;
>  }
> 
>  @Override
>  public boolean equals(Object o) {
>      return true;
>  }
> }
> ```
>
> - 即使两个A 对象通过 equals() 比较返回true,但HashSet 依然把他们当成 两个对象，即使两个 B 对象 的hashCode() 返回值相同，但HashSet 依然把他们当成两个对象。
> - 即也就是，当把一个对象放入HashSet 中时，如果需要重写该对象对应类的 equals() 方法，则也应该重写其 hashCode() 方法。规则是：如果两个对象通过 equals() 方法比较返回true,这两个对象的 hashCode 值也应该相同。
> - 如果两个对象通过euqals() 方法比较返回true,但这两个对象的 hashCode() 方法返回不同的hashCode 值时，这将导致HashSet 会把这两个对象保存在 Hash 表的不同位置，从而使两个对象都可以添加成功，这就与 Set 集合的规则冲突了。
> - 如果两个对象的 hashCode() 方法返回的 hasCode 值相同，但他们通过 equals() 方法比较返回false 时将更麻烦：因为两个对象的hashCode 值相同，HashSet 将试图 把它们保存在同一个位置，但又不行(否则将只剩下一个对象)，所以实际上会在这个位置用链式结构来保存多个对象；而HashSet 访问集合元素时也是根据元素的 hashCode 值来快速定位的，如果 hashSet 中两个以上的元素具有相同的 HashCode 值时，将会导致性能下降。

##### 简单说一下equals(Object obj)方法

> - 如果一个类没有重写 enquals(Object obj)方法，则等价于通过 == 比较两个对象，即比较的是对象在内存中的空间地址是否相等。
> - 如果重写了equals(Object ibj)方法，则根据重写的方法内容去比较相等，返回 true 则相等，false 则不相等。