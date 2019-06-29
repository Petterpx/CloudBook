# Java 集合（重点）

> java 集合大致可分为 Set,List,Queue和Map 四种体系。其中个set 代表无序，不可重复的集合，List 代表有序，重复的集合； 而 Mao 则代表具有映射关系的集合， Java5 又增加了 Queue 体系集合，代表一种队列集合实现。
>
> Java 集合就像一种容器，可以吧多个 对象 (实际上是对象的引用，但习惯上都称为对象 ) "丢进" 该容器中。 在 Java 5之前，Java 集合会丢失容器中 所有对象2的数据类型，把所有对象 都当成 Object 类型处理；从Java 5 增加泛型后，Java 集合可以记住容器中对象的数据类型，从而可以编写出更简洁，健壮的代码。

#### 集合和数组有什么不同？

> - 数组长度不可变，集合可变
> - 数组无法保存具有映射关系的数据
> - 数组即可以时基本类型的值，也可以是对象，而集合里只能保留对象

#### 简单对 Java 集合进行一个概述？

> Java 集合主要由两个接口派生而出：Collection 和 Map,Collection 和 Map 是 Java 集合框架的根接口，这两个接口又包含了一些子接口或实现类。
>
> ![img](http://upload-images.jianshu.io/upload_images/3985563-e7febf364d8d8235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>
> ![img](http://upload-images.jianshu.io/upload_images/3985563-06052107849a7603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>
> Map 实现类用于保存具有映射关系的数据，Map保存的每项数据都是 key-value对，也就是 由 key和 value 两个值组成。Map 里的key 是不可重复的，key 用户标识集合里的每项数据。

### Collection接口

> Collection 接口是 Set接口，Queue,List 的父接口，Collection 接口中定义了多种方法可供其子类实现，以实现数据操作。
>
> 
>
> - - | Modifier and Type        | Method and Description                                       |
>  | ------------------------ | ------------------------------------------------------------ |
> | `boolean`                | `add(E e)`  确保此集合包含指定的元素（可选操作）。           |
> | `boolean`                | `addAll(Collection<? extends E> c)`  将指定集合中的所有元素添加到此集合（可选操作）。 |
> | `void`                   | `clear()`  从此集合中删除所有元素（可选操作）。              |
> | `boolean`                | `contains(Object o)`  如果此集合包含指定的元素，则返回 `true` 。 |
> | `boolean`                | `containsAll(Collection<?> c)`  如果此集合包含指定 `集合`中的所有元素，则返回true。 |
> | `boolean`                | `equals(Object o)`  将指定的对象与此集合进行比较以获得相等性。 |
> | `int`                    | `hashCode()`  返回此集合的哈希码值。                         |
> | `boolean`                | `isEmpty()`  如果此集合不包含元素，则返回 `true` 。          |
> | `Iterator<E>`            | `iterator()`  返回此集合中的元素的迭代器。                   |
> | `default Stream<E>`      | `parallelStream()`   返回可能并行的 `Stream`与此集合作为其来源。 |
> | `boolean`                | `remove(Object o)`  从该集合中删除指定元素的单个实例（如果存在）（可选操作）。 |
> | `boolean`                | `removeAll(Collection<?> c)`  删除指定集合中包含的所有此集合的元素（可选操作）。 |
> | `default boolean`        | `removeIf(Predicate<? super E> filter)`  删除满足给定谓词的此集合的所有元素。 |
> | `boolean`                | `retainAll(Collection<?> c)`  仅保留此集合中包含在指定集合中的元素（可选操作）。 |
> | `int`                    | `size()`  返回此集合中的元素数。                             |
> | `default Spliterator<E>` | `spliterator()`   创建一个[`Spliterator`](../../java/util/Spliterator.html)在这个集合中的元素。 |
> | `default Stream<E>`      | `stream()`  返回以此集合作为源的顺序 `Stream` 。             |
> | `Object[]`               | `toArray()`  返回一个包含此集合中所有元素的数组。            |
> | `<T> T[]`                | `toArray(T[] a)`   返回包含此集合中所有元素的数组;  返回的数组的运行时类型是指定数组的运行时类型。 |

#### 使用Iterator遍历集合元素

> Iterator 接口是Java 集合框架的成员，但 它 与 Collection 系列，Map系列的集合不一样，Collection 系列集合，Map 系列集合主要用于盛装其他对象，而 Iterator 则主要用于遍历(即迭代访问) Collrction集合中的元素，Iterator 对象也被称为迭代器。
>
> - boolean hasNext()： 如果被迭代的集合元素还没有被遍历完，则返回 true
> - Object next(): 返回集合里的下一个元素。
> - void remove()：删除集合里上一次 next 方法返回的元素
> - void forEachRemaining(Consumer action) 这是Java8 为Iterator 新增的默认方法，该方法使用 Lambda 表达式来遍历集合元素。
>
> ```java
> public class MyClass {
> public static void main(String[] args) {
>  String[] data={"1","2","3","4"};
>  List<String> list=new ArrayList<String>(Arrays.asList(data));
>  Iterator it=list.iterator();
>  while (it.hasNext()){
>      String book= (String) it.next();
>      //返回集合里的下一个元素
>      it.next();
>      it.remove();
>      System.out.println(book);
>  }
>  list.forEach(a-> System.out.println(a));
> }
> }
> ```
>
> 当使用 Iterator 迭代访问 Collrction 集合元素时，Collrction 集合里的元素不能被改变，只有通过 Iteratoe 的 remove() 方法删除上一次next() 方法返回的集合元素才可以，否则将会引发 java.util.ConcurrentModeificationException 异常。
>
> ```java
> public class MyClass {
> public static void main(String[] args) {
>  String[] data={"1","2","3","4"};
>  List<String> list=new ArrayList<String>(Arrays.asList(data));
>  Iterator it=list.iterator();
>  while (it.hasNext()){
>      String book= (String) it.next();
>      //使用 Iterator迭代过程中，不可修改集合元素，下面代码引发异常
>      list.remove(book);
>      System.out.println(book);
>  }
> //        list.forEach(a-> System.out.println(a));
> }
> }
> ```
>
> Iterator 迭代器采用的是快速 失败(fail-fast) 机制，一旦在迭代过程中检测到该集合已经被修改(通常是程序中的其他线程修改)，程序立即引发 ConcurrenModificationException 异常，而不是现实修改后的结果，这样可以避免共享资源而引发的潜在问题。

### Set集合

> Set集合类似一个罐子，程序可以一次把多个对象 “丢进” Set集合，而Set 集合通常不能记住元素的添加顺序。Set 集合 和 Collection 基本相同，没有提供任何额外的方法。实际上Set 就是Collection，只是行为略有不同(Set不允许包含重复元素).
>
> Set集合不允许包含相同的元素，如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()方法返回false,且新元素不会被加入。



### List集合

> list集合代表一个元素有序，可重复的集合，集合中每个元素都有其相应的顺序索引。List 集合允许使用重复元素，可以通过索引来访问指定位置的集合元素。List 集合默认按元素的添加顺序 设置元素的索引。
>
> List 作为 Collection 接口的子接口，可以使用 Collection 接口里的全部方法。而且由于 List 是有序集合，因此 List 集合里增加了一些根据索引来操作集合元素的方法。
>
> > **void add(int index, Object element):** 在列表的指定位置插入指定元素（可选操作）。
> >
> > **boolean addAll(int index, Collection<? extends E> c) :** 将集合c 中的所有元素都插入到列表中的指定位置index处。
> >
> > **Object get(index):** 返回列表中指定位置的元素。
> >
> > **int indexOf(Object o):** 返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。
> >
> > **int lastIndexOf(Object o):** 返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。
> >
> > **Object remove(int index):** 移除列表中指定位置的元素。
> >
> > **Object set(int index, Object element):** 用指定元素替换列表中指定位置的元素。
> >
> > **List subList(int fromIndex, int toIndex):** 返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的所有集合元素组成的子集。
> >
> > **Object[] toArray():** 返回按适当顺序包含列表中的所有元素的数组（从第一个元素到最后一个元素）。
>
> 除此之外，Java 8还为List接口添加了如下两个默认方法。
>
> > **void replaceAll(UnaryOperator operator):** 根据operator指定的计算规则重新设置List集合的所有元素。
> >
> > **void sort(Comparator c):** 根据Comparator参数对List集合的元素排序。



### Queue集合

> Queue 用户模拟队列这种数据结构，队列通常是指“先进先出”(FIFO)的容器。队列的 头部 是在队列中存放时间最长的元素，队列的尾部是保存在队列中存放时间最短的元素。新元素插入 (offer) 到队列的尾部，访问元素 (poll) 操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。
>
> 
>
> - - |           | `add(E e)`  将指定的元素插入到此队列中，如果可以立即执行此操作，而不会违反容量限制， `true`在成功后返回  `IllegalStateException`如果当前没有可用空间，则抛出IllegalStateException。 |
>  | --------- | ------------------------------------------------------------ |
> | `E`       | `element()`  检索，但不删除，这个队列的头。                  |
> | `boolean` | `offer(E e)`  如果在不违反容量限制的情况下立即执行，则将指定的元素插入到此队列中。 |
> | `E`       | `peek()`  检索但不删除此队列的头，如果此队列为空，则返回 `null` 。 |
> | `E`       | `poll()`  检索并删除此队列的头，如果此队列为空，则返回 `null` 。 |
> | `E`       | `remove()`  检索并删除此队列的头。                           |



## Map集合

> Map集合保存具有映射关系的数据，因此Map 集合里保存着两组数，一组值用户保存Map 里的key,另一组值用户保存Map 里的 value, key和 value 都可以是任何 引用类型的 数据。Map 的 key 不允许重复，即同一个 Map 对象的任何两个 key 通过 equals 方法比较总是返回 false.
>
> 如下图所示，key和 value 之间存在单向一对一关系，即通过指定的 key,总能找到唯一的，确定的 value。从Map 中取出数据时，只要给出指定的 key,就可以取出对应的 value.
>
> ![img](http://upload-images.jianshu.io/upload_images/3985563-51f6c5278df941fe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Map集合与Set集合，List 集合的关系

##### 与Set集合的关系

> 如果把Map 里的所有key放在一起看，他们就组成了一个 Set 集合(所有的 key 没有顺序，key与key之间不能重复)，实际上 Map 确实包含了 一个 keySet() 方法，用户返回 Map 里所有key组成的Set集合。

##### 与List 集合的关系

> 如果把Map 里的所有value 放在一起来看，他们有非常类似于一个 List,元素与元素之间可以重复，每个元素可以根据索引来查找，只是Map 中索引不再使用整数值，而是 以另外一个对象作为索引。

##### 接口中的方法

> 
>
> - - | Modifier and Type     | Method and Description                                       |
>  | --------------------- | ------------------------------------------------------------ |
> | `void`                | `clear()`  从该地图中删除所有的映射（可选操作）。            |
> | `default V`           | `compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`  尝试计算指定键的映射及其当前映射的值（如果没有当前映射， `null` ）。 |
> | `default V`           | `computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)`  如果指定的键尚未与值相关联（或映射到 `null`  ），则尝试使用给定的映射函数计算其值，并将其输入到此映射中，除非 `null` 。 |
> | `default V`           | `computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`  如果指定的密钥的值存在且非空，则尝试计算给定密钥及其当前映射值的新映射。 |
> | `boolean`             | `containsKey(Object key)`  如果此映射包含指定键的映射，则返回 `true` 。 |
> | `boolean`             | `containsValue(Object value)`  如果此地图将一个或多个键映射到指定的值，则返回 `true` 。 |
> | `Set<Map.Entry<K,V>>` | `entrySet()`  返回此地图中包含的映射的[`Set`](../../java/util/Set.html)视图。 |
> | `boolean`             | `equals(Object o)`  将指定的对象与此映射进行比较以获得相等性。 |
> | `default void`        | `forEach(BiConsumer<? super K,? super V> action)`  对此映射中的每个条目执行给定的操作，直到所有条目都被处理或操作引发异常。 |
> | `V`                   | `get(Object key)`  返回到指定键所映射的值，或 `null`如果此映射包含该键的映射。 |
> | `default V`           | `getOrDefault(Object key, V defaultValue)`  返回到指定键所映射的值，或 `defaultValue`如果此映射包含该键的映射。 |
> | `int`                 | `hashCode()`  返回此地图的哈希码值。                         |
> | `boolean`             | `isEmpty()`  如果此地图不包含键值映射，则返回 `true` 。      |
> | `Set<K>`              | `keySet()`  返回此地图中包含的键的[`Set`](../../java/util/Set.html)视图。 |
> | `default V`           | `merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)`  如果指定的键尚未与值相关联或与null相关联，则将其与给定的非空值相关联。 |
> | `V`                   | `put(K key, V value)`  将指定的值与该映射中的指定键相关联（可选操作）。 |
> | `void`                | `putAll(Map<?  extends K,?  extends V> m)`  将指定地图的所有映射复制到此映射（可选操作）。 |
> | `default V`           | `putIfAbsent(K key, V value)`  如果指定的键尚未与某个值相关联（或映射到 `null` ）将其与给定值相关联并返回  `null` ，否则返回当前值。 |
> | `V`                   | `remove(Object key)`  如果存在（从可选的操作），从该地图中删除一个键的映射。 |
> | `default boolean`     | `remove(Object key, Object value)`  仅当指定的密钥当前映射到指定的值时删除该条目。 |
> | `default V`           | `replace(K key, V value)`  只有当目标映射到某个值时，才能替换指定键的条目。 |
> | `default boolean`     | `replace(K key, V oldValue, V newValue)`  仅当当前映射到指定的值时，才能替换指定键的条目。 |
> | `default void`        | `replaceAll(BiFunction<? super K,? super V,? extends V> function)`  将每个条目的值替换为对该条目调用给定函数的结果，直到所有条目都被处理或该函数抛出异常。 |
> | `int`                 | `size()`  返回此地图中键值映射的数量。                       |
> | `Collection<V>`       | `values()`  返回此地图中包含的值的[`Collection`](../../java/util/Collection.html)视图。 |

Map中还包括了一个内部类Entry,该类封装了一个 key-value对，Entry包含如下方法。



- - | Modifier and Type                                            | Method and Description                                       |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | `static <K extends Comparable<? super  K>,V>Comparator<Map.Entry<K,V>>` | `comparingByKey()`   返回一个比较[器](../../java/util/Map.Entry.html) ，按键自然顺序比较[`Map.Entry`](../../java/util/Map.Entry.html) 。 |
    | `static <K,V> Comparator<Map.Entry<K,V>>`                    | `comparingByKey(Comparator<? super  K> cmp)`  返回一个比较器，比较[`Map.Entry`](../../java/util/Map.Entry.html)按键使用给定的[`Comparator`](../../java/util/Comparator.html) 。 |
    | `static <K,V extends Comparable<? super V>>Comparator<Map.Entry<K,V>>` | `comparingByValue()`   返回一个比较器，比较[`Map.Entry`](../../java/util/Map.Entry.html)的自然顺序值。 |
    | `static <K,V> Comparator<Map.Entry<K,V>>`                    | `comparingByValue(Comparator<? super  V> cmp)`  返回一个比较[器](../../java/util/Map.Entry.html) ，使用给定的[`Comparator`](../../java/util/Comparator.html)比较[`Map.Entry`](../../java/util/Map.Entry.html)的值。 |
    | `boolean`                                                    | `equals(Object o)`  将指定的对象与此条目进行比较以获得相等性。 |
    | `K`                                                          | `getKey()`  返回与此条目相对应的键。                         |
    | `V`                                                          | `getValue()`  返回与此条目相对应的值。                       |
    | `int`                                                        | `hashCode()`  返回此映射条目的哈希码值。                     |
    | `V`                                                          | `setValue(V value)`  用指定的值替换与该条目相对应的值（可选操作）。 |

Map 集合最典型的用法就是成对的添加，删除 key-value 对，然后就是判断该Map 中是否包含 指定 key,是否包含指定 value,也可以通过Map提供的 keySet() 方法获取所有 key 组成的集合，然后使用 foreach 循环来遍历Map 的所有 key,根据key 即可遍历所有的 value.



## ArrayList

> 以数组实现。节约空间，但数组有容量限制。超出限制是会增加50% 的容量，用 System.arraycopy() 复制到新数组，因此最好能给出数组大小的预估值，默认第一次插入元素时创建大小为10的数组。
>
> 按数组下标访问元素——get(i)/set(i,e) 的性能很高，这是数组的基本优势
>
> 直接在数组的末尾加入元素——add(e) 的性能也高，但如下标插入，删除元素——add(i,e),remove(i),remove(e),则要用System.arrycopy() 来移动部分受影响的元素，性能就变差了，这是基本劣势。

ArrayList 是一个相对来说比较简单的数据结构，最重要的一点就是它的自动扩容，可以认为就是我们常说的 “动态数组”

> 看下面例子
>
> ```java
> ArrayList<String> list = new ArrayList<String>();
> list.add("语文: 99");
> list.add("数学: 98");
> list.add("英语: 100");
> list.remove(0);
> ```
>
> 在执行这四条语句时，是这么变化的：![arraylist](https://cloud.githubusercontent.com/assets/1736354/6993037/5d4ba306-db19-11e4-85fb-61b0154d0d96.png)
>
> 其中，add 操作可以理解为直接将数组的内容置位， remove 操作可以理解为 删除 index 为0 的节点，并将后面元素移到0 处。



#### add函数

当我们在ArrayList中增加元素的时候，会使用 add 函数。它会将元素放到末尾。具体实现如下。

```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;		//elementData存储arraylist元素的数组
        return true;
    }
```

我们可以看到它的实现其实最核心内容就是  ensureCapacityInternal ，这个函数其实就是自动扩容机制的核心。

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //复制到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

也就是说，当增加数据的时候，如果 ArrayList的大已经不再满足需求是，那么就将数组变为原长度的 1.5倍，之后的操作就是把 老的 数组拷到新的数组里面。例如，默认数组大小是10，也就是说当我们 add 10个元素之后，再进行一次 add 时，就会发生自动扩容。

![arraylistadd](https://cloud.githubusercontent.com/assets/1736354/6993129/e892246e-db1c-11e4-9ae8-f9719688a1ca.png)

#### set和get函数

> Array 的set和 get函数先做index检查，然后执行赋值或访问操作
>
> ```java
> public E set(int index, E element) {
>     if (index >= size)
>         throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
> 
>     E oldValue = (E) elementData[index];
>     elementData[index] = element;
>     return oldValue;
> }
>   public E get(int index) {
>         if (index >= size)
>             throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
> 
>         return (E) elementData[index];
>     }
> ```

#### remove 函数

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    //计算指定下标后面的长度，便于下一步将后面的元素前移
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 把后面的往前移
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 把最后的置null
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}


//第二种重载
  public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                //判断是否存在该元素
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
  private void fastRemove(int index) {
        modCount++;
      	//和上面的remove机制一样，先把指定下标后的长度拿到
        int numMoved = size - index - 1;
        if (numMoved > 0)
            //将数组往前移一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
      	//将最后一位置空，然后让垃圾回收机制去回收
        elementData[--size] = null; // clear to let GC do its work
    }
```

```
综上所述，在用户向ArrayList追加对象时，Java总是要先计算容量（Capacity）是否适当，若容量不足则把原数组拷贝到以指定容量为长度创建的新数组内，并对原数组变量重新赋值，指向新数组。在这同时，size进行自增``1``。在删除对象时，先使用拷贝方法把指定index后面的对象前移``1``位（如果有的话），然后把空出来的位置置``null``，交给Junk收集器销毁，size自减``1``，即完成了。
```

#### 问答

##### 简单讲一下ArrayList的实现原理？

ArrayList内部使用数组实现，为了节约空间。但数组有容量限制，超出限制会增加50%的容量，然后使用 Syste.arraycopy() 复制到新数组，因此我们最好能给出数组的大小预估值。默认第一次插入元素时创建大小为10的数组。

因为它按数组下标访问元素，所以get与set性能很高。但是如果remove()，或者下标插入元素的话效率就比较慢了，因为它需要用 Syustem.arraycopy() 来移动部分受影响的元素，导致性能下降，这是基本劣势。

##### 说一下add和get,及它的remove方法吧？

1. ##### add（）

   当我们在ArrayList中增加元素的时候，会使用 add 函数。它会将元素放到末尾。其中最核心的内容就是ensureCapacityInternal ，也就是自动扩容机制的核心，增加数据时，如果 ArrayList 的大小以及不再满足需求，那么就将数组变为原长度的1.5倍,之后的操作就是把老的数组拷到新的数组里面,并对原数组变量重新赋值，指向新数组。

2. ##### set() 和 get()

   Array 的set和get会先做index 检查，然后执行赋值或访问操作,内部的操作就是数组的直接赋值和取值

3. ##### remove()

   在删除对象时，先使用拷贝方法把指定index后面的对象前移``1``位（如果有的话），然后把空出来的位置置``null``，交给Junk收集器销毁，size自减``1``，即完成了。





## LinkedList

> 以双向链表实现。链表无容量限制。但双向链表本身使用了更多空间，也需要额外的链表指针操作。
>
> 按下标访问元素——get(i)/set(i,e)  要悲剧的遍历链表，将指针移动到位(如果 i>数组大小的一半，会从末尾移起)
>
> 插入，删除元素时修改前后节点的 指针即可，但还是要遍历部分链表的指针才能移动到下标所指的位置，只有在链表两头的操作 ——add()，addFitst(),removeLast() 或用iterator() 上的 remove()能省略掉指针的移动。
>
> LinkedList 是一个简单的数据机构，与ArrayList 不同的是，它是基于链表实现的。

```java
LinkedList<String> list = new LinkedList<String>();
list.add("语文: 1");
list.add("数学: 2");
list.add("英语: 3");
```

![linkedlist](https://cloud.githubusercontent.com/assets/1736354/6997435/92fab224-dbed-11e4-932a-4c5593a2abb7.png)

#### set和get函数

```java
public E set(int index, E element) {
	//判断边界
    checkElementIndex(index);
    //链表
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}

    public E get(int index) {
        checkElementIndex(index);
        //返回指定位置元素
        return node(index).item;
    }
```

这两个函数都调用了 node 函数，该函数会以 O(n/2)的性能,关于算法效率问题，[参考](<https://www.jianshu.com/p/59d09b9cee58>)

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

就是判断index 是在前半区间还时后半区间，如果在前半区间就从 head搜索，而在后半区间就从 tail 搜索。而不是一直从头到尾的搜索。这也就是 节点访问复杂度 由O(n)变为 O(n/2);





## HashMap实现原理

hashMap是基于哈希表的Map 接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是他不保证顺序的恒久不变。

#### HashMap的数据结构

在java 编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针(引用)，所有的数据结构都可以用这两个基本结构来构造的，HashMap 也不例外。HashMap实际上是一个 “链表散列” 的数据结构，即数组和链表的结合体。



#### 概要

HashMap底层就是一个数组结构，数组的每一项又是一个链表，当新建一个HashMap 的时候，就会初始化一个数组。

```java
transient Entry[] table;  
static class Entry<K,V> implements Map.Entry<K,V> {  
final K key;  
V value;  
Entry<K,V> next;  
final int hash;  
……  
}  
```

可以看出，Entry就是数组中的元素，每个Map.Entry其实就是一个 key-value对，它持有一个指向下一个元素的引用，这就构成了链表。

对于HashMap，HashSet来说，他们采用hash 算法来决定 Map中 key的存储，并通过hash算法来增加key 集合 的大小。

hash表中可以存储元素的位置被称为 “桶(bucket)”，在通常情况下，单个“桶”里存储一个元素，此时有最好的性能；hash 算法可以根据 hashCode 值计算出"桶"的存储位置（[hashCode()的具体解释请看我篇博客](<https://blog.csdn.net/petterp/article/details/89043847>)），接着从桶中取出元素。但hash 表的状态是 open的：在发生 “hash” 冲突的情况下，单个桶 会存储多个元素，这些元素以链表形式存储，如下图。

![1554449704489](C:\Users\Pettepr\AppData\Roaming\Typora\typora-user-images\1554449704489.png)

#### 重要的参数

> - Capacity(容量)：Capacity 就是 bucket的大小，指的也就是HashMap集合初始化的时候自身的容量。可以在构造方法中指定；如果不指定的话，总容量默认值为16.需要注意的是初始容量必须是2的幂次方。
> - loadfactor(负载因子)：Load factor就是bucket填满程度的最大比例。当 HashMap的总容量到达一定值得时候，HashMap会实施扩容，负责因子也可以通过构造方法中指定，默认值为0.75.也就是说，比如你现在有一个 HashMap 的初始容量为10，那么扩容的阈值就是0.75*10=7.5，也就是说，在你打算存入第8个值得时候，HashMap 会先执行扩容。
> - threshold: 扩容阈值，即扩容阈值=HashMap 总容量*负载因子，当前 HashMao 的容量大于或等于扩容阈值的时候就会去执行扩容。扩容的容量为当前 HashMap 总容量的两倍。
> - size(尺寸)：当前 hash 表中记录的数量
> - initial capacity(初始化容量)：创建hash表时桶的数量，可以在构造器中指定初始化容量。
>
> 



#### put方法

先简单讲一下大概的思路：

1. 对key的hashCode() 做hash,然后在计算index
2. 如果没有冲突，则则直接放到bucket(桶)里，
3. 如果冲突了，以链表的形式存在buckets后
4. 如果冲突添加后导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树
5. 如果节点已经存在就替换 old value（保证key的唯一性）
6. 如果bucket满了（超过 负载因子），就要resize（扩容）

```java
public V put(K key, V value) {
    //对key的hashCode() 做hash
     return putVal(hash(key), key, value, false, true);
 }
```

再看它的putVal方法

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
            boolean evict) {
 Node<K,V>[] tab; Node<K,V> p; int n, i;
 //tab为空则创建
 if ((tab = table) == null || (n = tab.length) == 0)
     n = (tab = resize()).length;
 //计算index,并对null做处理
 if ((p = tab[i = (n - 1) & hash]) == null)
     tab[i] = newNode(hash, key, value, null);
 else {
     Node<K,V> e; K k;
     //判断如果节点存在
     if (p.hash == hash &&
         ((k = p.key) == key || (key != null && key.equals(k))))
         e = p;
     //该链为树
     else if (p instanceof TreeNode)
         e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
     //该链为链表
     else {
         for (int binCount = 0; ; ++binCount) {
             if ((e = p.next) == null) {
                 p.next = newNode(hash, key, value, null);
                 if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                     treeifyBin(tab, hash);
                 break;
             }
             if (e.hash == hash &&
                 ((k = e.key) == key || (key != null && key.equals(k))))
                 break;
             p = e;
         }
     }
     //写入
     if (e != null) { // existing mapping for key
         V oldValue = e.value;
         if (!onlyIfAbsent || oldValue == null)
             e.value = value;
         afterNodeAccess(e);
         return oldValue;
     }
 }
 ++modCount;
 //超过 load factor*current capacity(负载因子*hashmap容量)，resize
 if (++size > threshold)
     resize();
 afterNodeInsertion(evict);
 return null;
}
```



#### get 方法

get的思路如下：

1. bucket里的第一个节点，直接命中（即直接返回）
2. 如果有冲突，则通过 key.equals(k) 去查找对应的 entry

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //直接命中
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //未命中
        if ((e = first.next) != null) {
            //在树中找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //在链表中找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### **hash函数的实现**

在get 和 put 的过程中，计算下标时，先对hashCode 进行hash 操作，然后在通过hash 值进行下一步计算下标。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

在获取 HashMap 的元素时，基本分两步：

1. 首先根据hashCode()做hash,然后确定 bucket 的 index
2. 如果bucket 的节点的key 不是我们需要的，则通过key.equals()在链中找。

   在Java8 之前的实现中是用链表解决冲突的，在产生碰撞的情况下，进行get时，两步的时间复杂度是 O(1)+O(n).因此，当碰撞很厉害的时候，n很大，O(n)的速度显示是影响速度的。

   因此在Java8中，利用红黑树替换链表，这样复杂符就变成了了 O(1)+O(logn)了，这样在n很大的时候，能够比较理想的解决这个问题。



#### **resize的实现**

当put时，如果发现目前的bucket占用程度已经超过了 负载因子 所希望的比例，那么就会发生 resize,在resize 的过程中，简单的说就是把 bucket 扩充为2倍，之后重新计算index，把节点在放到新的 bucket中。

怎么理解呢？例如我们从16扩展为32时，具体的变化如下所示：

![rehash](https://cloud.githubusercontent.com/assets/1736354/6958256/ceb6e6ac-d93b-11e4-98e7-c5a5a07da8c4.png)

因此元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![resize](https://cloud.githubusercontent.com/assets/1736354/6958301/519be432-d93c-11e4-85bb-dff0a03af9d3.png)

因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。可以看看下图为16扩充为32的resize示意图：

![resize16-32](https://cloud.githubusercontent.com/assets/1736354/6958677/d7acbad8-d941-11e4-9493-2c5e69d084c0.png)

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。

```java
final Node<K,V>[] resize() {
 Node<K,V>[] oldTab = table;
 int oldCap = (oldTab == null) ? 0 : oldTab.length;
 int oldThr = threshold;
 int newCap, newThr = 0;
 if (oldCap > 0) {
     // 超过最大值就不再扩充了，就只好随你碰撞去吧
     if (oldCap >= MAXIMUM_CAPACITY) {
         threshold = Integer.MAX_VALUE;
         return oldTab;
     }
     // 没超过最大值，就扩充为原来的2倍
     else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
              oldCap >= DEFAULT_INITIAL_CAPACITY)
         newThr = oldThr << 1; // double threshold
 }
 else if (oldThr > 0) // initial capacity was placed in threshold
     newCap = oldThr;
 else {               // zero initial threshold signifies using defaults
     newCap = DEFAULT_INITIAL_CAPACITY;
     newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
 }
 // 计算新的resize上限
 if (newThr == 0) {
     float ft = (float)newCap * loadFactor;
     newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
               (int)ft : Integer.MAX_VALUE);
 }
 threshold = newThr;
 @SuppressWarnings({"rawtypes","unchecked"})
     Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
 table = newTab;
 if (oldTab != null) {
     // 把每个bucket都移动到新的buckets中
     for (int j = 0; j < oldCap; ++j) {
         Node<K,V> e;
         if ((e = oldTab[j]) != null) {
             oldTab[j] = null;
             if (e.next == null)
                 newTab[e.hash & (newCap - 1)] = e;
             else if (e instanceof TreeNode)
                 ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
             else { // preserve order
                 Node<K,V> loHead = null, loTail = null;
                 Node<K,V> hiHead = null, hiTail = null;
                 Node<K,V> next;
                 do {
                     next = e.next;
                     // 原索引
                     if ((e.hash & oldCap) == 0) {
                         if (loTail == null)
                             loHead = e;
                         else
                             loTail.next = e;
                         loTail = e;
                     }
                     // 原索引+oldCap
                     else {
                         if (hiTail == null)
                             hiHead = e;
                         else
                             hiTail.next = e;
                         hiTail = e;
                     }
                 } while ((e = next) != null);
                 // 原索引放到bucket里
                 if (loTail != null) {
                     loTail.next = null;
                     newTab[j] = loHead;
                 }
                 // 原索引+oldCap放到bucket里
                 if (hiTail != null) {
                     hiTail.next = null;
                     newTab[j + oldCap] = hiHead;
                 }
             }
         }
     }
 }
 return newTab;
}
```



#### 简要问答

**1. 什么时候会使用HashMap？他有什么特点？**
是基于Map接口的实现，存储键值对时，它可以接收null的键值，是非同步的，HashMap存储着Entry(hash, key, value, next)对象。

**2. 你知道HashMap的工作原理吗？**
通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过`Load Facotr`则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

**3. 你知道get和put的原理吗？equals()和hashCode()的都有什么作用？**
通过对key的hashCode()进行hashing，并计算下标( `(n-1) & hash`)，从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中去查找对应的节点

**4. 你知道hash的实现吗？为什么要这样实现？**
在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：`(h = k.hashCode()) ^ (h >>> 16)`，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

**5. 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？**
如果超过了负载因子(默认**0.75**)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。





## TreeMap

> TreeMap 是一个**有序的key-value集合**，它是通过[红黑树](http://www.cnblogs.com/skywang12345/p/3245399.html)实现的。
> TreeMap **继承于AbstractMap**，所以它是一个Map，即一个key-value集合。
> TreeMap 实现了NavigableMap接口，意味着它**支持一系列的导航方法。**比如返回有序的key集合。
> TreeMap 实现了Cloneable接口，意味着**它能被克隆**。
> TreeMap 实现了java.io.Serializable接口，意味着**它支持序列化**。
>
> TreeMap基于**红黑树（Red-Black tree）实现**。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。
> TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。
> 另外，TreeMap是**非同步**的。 它的iterator 方法返回的**迭代器是fail-fastl**的。
>
> 
>
> HashMap 不保证数据有序，LinkedHashMap 保证数据可以保持插入顺序，而如果我们希望 Map 可以 **保持key 的大小顺序** 时，我们就需要利用 TreeMap 了。

```java
TreeMap<Integer, String> tmap = new TreeMap<Integer, String>();
tmap.put(1, "语文");
tmap.put(3, "英语");
tmap.put(2, "数学");
tmap.put(4, "政治");
tmap.put(5, "历史");
tmap.put(6, "地理");
tmap.put(7, "生物");
tmap.put(8, "化学");
for(Entry<Integer, String> entry : tmap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

![treemap](https://cloud.githubusercontent.com/assets/1736354/7041463/05ee676a-de0c-11e4-9412-4c6964931e43.png)

它上面使用了 红黑树，使用红黑树的好处就是 **使得树具有不错的平衡性**,这样操作的速度就可以达到 log(n)的水平了

#### Put 函数

如果存在的话，old value倍替换，如果不存在的话，则新添一个节点，然后对红黑树的平衡操作。

```java
public V put(K key, V value) {
    TreeMapEntry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check
        root = new TreeMapEntry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    TreeMapEntry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    //如果该节点存在，则替换值直接返回
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    //如果该节点未存在，则新建
    TreeMapEntry<K,V> e = new TreeMapEntry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    
    //红黑树平衡调整
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```



#### get函数

get函数则相对来说比较简单，移 log(n)的复杂度进行 get

```java
public V get(Object key) {
    TreeMapEntry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

 final TreeMapEntry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        TreeMapEntry<K,V> p = root;
     //按照二叉树搜索的方式进行搜索
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
```



#### successor 后继

TreeMap 是如何保证迭代输出是有序的呢？其实从宏观上来讲，就相当于树的中序遍历(LDR) ，我们先看一下迭代输出的步骤

```java
for(Entry<Integer, String> entry : tmap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

根据[The enhanced for statement](http://docs.oracle.com/javase/specs/jls/se8/html/jls-14.html#jls-14.14.2)，for语句会做如下转换为：

```java
for(Iterator<Map.Entry<String, String>> it = tmap.entrySet().iterator() ; tmap.hasNext(); ) {
    Entry<Integer, String> entry = it.next();
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

在 **it.next()** 的调用中 会使用 nextEntry 调用 successor 这个是过的后继的重点，具体实现如下。

```java
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        // 有右子树的节点，后继节点就是右子树的“最左节点”
        // 因为“最左子树”是右子树的最小节点
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        // 如果右子树为空，则寻找当前节点所在左子树的第一个祖先节点
        // 因为左子树找完了，根据LDR该D了
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        // 保证左子树
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

这个是中序遍历。

> a. 空节点，没有后继
>
> b. 有右子树的节点，后继就是右子树的 最左节点
>
> c. 无右子树的节点，后继就是该节点所在左子树的第一个祖先节点

**有右子树的节点**，节点的下一个节点，肯定在右子树中，而右子树中“最左”的那个节点则是右子树中最小的一个，那么当然是**右子树的“最左节点”**，就好像下图所示：

![treemap1](https://cloud.githubusercontent.com/assets/1736354/7045283/652d00c4-de2f-11e4-8475-1a2f46afc380.png)

**无右子树的节点**，先找到这个节点所在的左子树(右图)，那么这个节点所在的左子树的父节点(绿色节点)，就是下一个节点。

![treemap2](https://cloud.githubusercontent.com/assets/1736354/7045284/68279686-de2f-11e4-8310-c9f76b3f52ab.png)





## LinkedHashMap

LinkedHashMap是Hash表 和 链表的实现，并且依靠着双向链表保证了迭代顺序是插入的顺序。

LinledHashMap 需要维护元素的插入顺序，因此性能略低于HashMap 的性能，但因为它以链表来维护内部顺序，所以在迭代访问 Map 里的全部元素时将有较好的性能。



#### 三个重点实现的函数

```Java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

LinkedHashMap 继承与HashMap,因此也重新实现了这三个函数，顾名思义这三个函数的作用分别是：节点访问，节点插入后，节点移除后做的一些事情

**afterNodeAccess函数**

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMapEntry<K,V> last;
    //如果定义了 accessOrder，那么就保证最近访问节点到最后。
    if (accessOrder && (last = tail) != e) {
        LinkedHashMapEntry<K,V> p =
            (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

就是说 在进行 put 之后就算是对节点的访问了，那么这个时候就会更新链表，把最近访问的放到最后，保证链表。

**afterNodeInsertion**函数

```Java
void afterNodeRemoval(Node<K,V> e) { // unlink
    // 从链表中移除节点
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

这个函数是在移除节点后调用的，就是将节点从双向链表删除。





## 操作集合的工具类：Collections

Java 提供了一个操作 Set，List和 Map 等集合的工具类 ：Collections,该工具类里提供了大量方法对集合元素进行排序，查询，修改等操作。还踢红了将集合对象设置为不可变，对集合对象实现同步控制等方法。



- - | Modifier and Type                                     | Method and Description                                       |
    | ----------------------------------------------------- | ------------------------------------------------------------ |
    | `static <T> boolean`                                  | `addAll(Collection<? super T> c,  T... elements)`  将所有指定的元素添加到指定的集合。 |
    | `static <T> Queue<T>`                                 | `asLifoQueue(Deque<T> deque)`  返回[`Deque`](../../java/util/Deque.html)作为先进先出（ [Lifo](../../java/util/Queue.html) ） [`Queue`的视图](../../java/util/Queue.html)  。 |
    | `static <T> int`                                      | `binarySearch(List<?  extends Comparable<? super T>> list,  T key)`  使用二叉搜索算法搜索指定对象的指定列表。 |
    | `static <T> int`                                      | `binarySearch(List<?  extends T> list, T key, Comparator<? super T> c)`   使用二叉搜索算法搜索指定对象的指定列表。 |
    | `static <E> Collection<E>`                            | `checkedCollection(Collection<E> c, 类<E> type)`  返回指定集合的动态类型安全视图。 |
    | `static <E> List<E>`                                  | `checkedList(List<E> list, 类<E> type)`  返回指定列表的动态类型安全视图。 |
    | `static <K,V> Map<K,V>`                               | `checkedMap(Map<K,V> m, 类<K> keyType, 类<V> valueType)`  返回指定地图的动态类型安全视图。 |
    | `static <K,V> NavigableMap<K,V>`                      | `checkedNavigableMap(NavigableMap<K,V> m, 类<K> keyType, 类<V> valueType)`  返回指定可导航地图的动态类型安全视图。 |
    | `static <E> NavigableSet<E>`                          | `checkedNavigableSet(NavigableSet<E> s, 类<E> type)`  返回指定的可导航集的动态类型安全视图。 |
    | `static <E> Queue<E>`                                 | `checkedQueue(Queue<E> queue, 类<E> type)`  返回指定队列的动态类型安全视图。 |
    | `static <E> Set<E>`                                   | `checkedSet(Set<E> s, 类<E> type)`  返回指定集合的动态类型安全视图。 |
    | `static <K,V> SortedMap<K,V>`                         | `checkedSortedMap(SortedMap<K,V> m, 类<K> keyType, 类<V> valueType)`  返回指定排序映射的动态类型安全视图。 |
    | `static <E> SortedSet<E>`                             | `checkedSortedSet(SortedSet<E> s, 类<E> type)`  返回指定排序集的动态类型安全视图。 |
    | `static <T> void`                                     | `copy(List<?  super T> dest, List<? extends T> src)`  将所有元素从一个列表复制到另一个列表中。 |
    | `static boolean`                                      | `disjoint(Collection<?> c1, Collection<?> c2)`  如果两个指定的集合没有共同的元素，则返回 `true` 。 |
    | `static <T> Enumeration<T>`                           | `emptyEnumeration()`   返回没有元素的枚举。                  |
    | `static <T> Iterator<T>`                              | `emptyIterator()`   返回没有元素的迭代器。                   |
    | `static <T> List<T>`                                  | `emptyList()`   返回空列表（immutable）。                    |
    | `static <T> ListIterator<T>`                          | `emptyListIterator()`   返回没有元素的列表迭代器。           |
    | `static <K,V> Map<K,V>`                               | `emptyMap()`  返回空的地图（不可变）。                       |
    | `static <K,V> NavigableMap<K,V>`                      | `emptyNavigableMap()`   返回空导航地图（不可变）。           |
    | `static <E> NavigableSet<E>`                          | `emptyNavigableSet()`   返回一个空导航集（immutable）。      |
    | `static <T> Set<T>`                                   | `emptySet()`  返回一个空集（immutable）。                    |
    | `static <K,V> SortedMap<K,V>`                         | `emptySortedMap()`   返回空的排序映射（immutable）。         |
    | `static <E> SortedSet<E>`                             | `emptySortedSet()`   返回一个空的排序集（immutable）。       |
    | `static <T> Enumeration<T>`                           | `enumeration(Collection<T> c)`  返回指定集合的枚举。         |
    | `static <T> void`                                     | `fill(List<?  super T> list, T obj)`  用指定的元素代替指定列表的所有元素。 |
    | `static int`                                          | `frequency(Collection<?> c, Object o)`  返回指定集合中与指定对象相等的元素数。 |
    | `static int`                                          | `indexOfSubList(List<?> source, List<?> target)`  返回指定源列表中指定目标列表的第一次出现的起始位置，如果没有此类事件，则返回-1。 |
    | `static int`                                          | `lastIndexOfSubList(List<?> source, List<?> target)`  返回指定源列表中指定目标列表的最后一次出现的起始位置，如果没有此类事件则返回-1。 |
    | `static <T> ArrayList<T>`                             | `list(Enumeration<T> e)`  返回一个数组列表，其中包含由枚举返回的顺序由指定的枚举返回的元素。 |
    | `static <T extends Object & Comparable<? super  T>>T` | `max(Collection<? extends  T> coll)`  根据其元素的 *自然顺序*返回给定集合的最大元素。 |
    | `static <T> T`                                        | `max(Collection<? extends T> coll,  Comparator<? super  T> comp)`  根据指定的比较器引发的顺序返回给定集合的最大元素。 |
    | `static <T extends Object & Comparable<? super  T>>T` | `min(Collection<? extends  T> coll)`  根据其元素的 *自然顺序*返回给定集合的最小元素。 |
    | `static <T> T`                                        | `min(Collection<? extends T> coll,  Comparator<? super  T> comp)`  根据指定的比较器引发的顺序返回给定集合的最小元素。 |
    | `static <T> List<T>`                                  | `nCopies(int n,  T o)`  返回由指定对象的 `n`副本组成的不可变列表。 |
    | `static <E> Set<E>`                                   | `newSetFromMap(Map<E,Boolean> map)`  返回由指定地图支持的集合。 |
    | `static <T> boolean`                                  | `replaceAll(List<T> list, T oldVal,  T newVal)`  将列表中一个指定值的所有出现替换为另一个。 |
    | `static void`                                         | `reverse(List<?> list)`  反转指定列表中元素的顺序。          |
    | `static <T> Comparator<T>`                            | `reverseOrder()`   返回一个比较器，它对实现 `Comparable`接口的对象集合施加了  *自然排序*的相反。 |
    | `static <T> Comparator<T>`                            | `reverseOrder(Comparator<T> cmp)`  返回一个比较器，它强制指定比较器的反向排序。 |
    | `static void`                                         | `rotate(List<?> list, int distance)`  将指定列表中的元素旋转指定的距离。 |
    | `static void`                                         | `shuffle(List<?> list)`  使用默认的随机源随机排列指定的列表。 |
    | `static void`                                         | `shuffle(List<?> list, Random rnd)`  使用指定的随机源随机排列指定的列表。 |
    | `static <T> Set<T>`                                   | `singleton(T o)`   返回一个只包含指定对象的不可变集。        |
    | `static <T> List<T>`                                  | `singletonList(T o)`   返回一个只包含指定对象的不可变列表。  |
    | `static <K,V> Map<K,V>`                               | `singletonMap(K key,  V value)`  返回一个不可变的地图，只将指定的键映射到指定的值。 |
    | `static <T extends Comparable<? super  T>>void`       | `sort(List<T> list)`  根据其元素的[natural ordering](../../java/lang/Comparable.html)对指定的列表进行排序。 |
    | `static <T> void`                                     | `sort(List<T> list, Comparator<? super T> c)`   根据指定的比较器引起的顺序对指定的列表进行排序。 |
    | `static void`                                         | `swap(List<?> list, int i, int j)`  交换指定列表中指定位置的元素。 |
    | `static <T> Collection<T>`                            | `synchronizedCollection(Collection<T> c)`  返回由指定集合支持的同步（线程安全）集合。 |
    | `static <T> List<T>`                                  | `synchronizedList(List<T> list)`  返回由指定列表支持的同步（线程安全）列表。 |
    | `static <K,V> Map<K,V>`                               | `synchronizedMap(Map<K,V> m)`  返回由指定地图支持的同步（线程安全）映射。 |
    | `static <K,V> NavigableMap<K,V>`                      | `synchronizedNavigableMap(NavigableMap<K,V> m)`  返回由指定的可导航地图支持的同步（线程安全）可导航地图。 |
    | `static <T> NavigableSet<T>`                          | `synchronizedNavigableSet(NavigableSet<T> s)`  返回由指定的可导航集支持的同步（线程安全）可导航集。 |
    | `static <T> Set<T>`                                   | `synchronizedSet(Set<T> s)`  返回由指定集合支持的同步（线程安全）集。 |
    | `static <K,V> SortedMap<K,V>`                         | `synchronizedSortedMap(SortedMap<K,V> m)`  返回由指定的排序映射支持的同步（线程安全）排序映射。 |
    | `static <T> SortedSet<T>`                             | `synchronizedSortedSet(SortedSet<T> s)`  返回由指定的排序集支持的同步（线程安全）排序集。 |
    | `static <T> Collection<T>`                            | `unmodifiableCollection(Collection<? extends  T> c)`  返回指定集合的不可修改视图。 |
    | `static <T> List<T>`                                  | `unmodifiableList(List<?  extends T> list)`  返回指定列表的不可修改视图。 |
    | `static <K,V> Map<K,V>`                               | `unmodifiableMap(Map<?  extends K,? extends V> m)`  返回指定地图的不可修改视图。 |
    | `static <K,V> NavigableMap<K,V>`                      | `unmodifiableNavigableMap(NavigableMap<K,? extends  V> m)`  返回指定可导航地图的不可修改视图。 |
    | `static <T> NavigableSet<T>`                          | `unmodifiableNavigableSet(NavigableSet<T> s)`  返回指定的可导航集合的不可修改的视图。 |
    | `static <T> Set<T>`                                   | `unmodifiableSet(Set<?  extends T> s)`  返回指定集合的不可修改视图。 |
    | `static <K,V> SortedMap<K,V>`                         | `unmodifiableSortedMap(SortedMap<K,? extends  V> m)`  返回指定排序映射的不可修改视图。 |
    | `static <T> SortedSet<T>`                             | `unmodifiableSortedSet(SortedSet<T> s)`  返回指定排序集的不可修改视图。 |

#### 同步控制

Collections 类中提供了多个 synchronizedXxx()方法，该方法可以将制定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题。

Java 中常用的集合框架中的实现类 HashSet，TreeSet，ArrayList,ArrayDeque，LinkedList ,HashMaP和 TreeMap 都是线程不安全的。

```java
public class Test {
    public static void main(String[] args) {
        //下面创建了4个线程安全的集合对象
        Collection c=Collections.synchronizedCollection(new ArrayList());
        List list=Collections.synchronizedList(new ArrayList());
        Map m=Collections.synchronizedMap(new HashMap());
    }
}
```



#### 设置不可变集合

Collections 提供了如下3类方法来返回一个不可变的集合

1. emptyXxx(): 返回一个空的，不可变的集合对象，此处的集合即可以是 List，也可以是 SortedSet,Set,还可以是Map,SortedMap等。

2. singletionXxx() ： 返回一个只包含指定对象(只有一个或一项元素)的，不可变的集合对象，此处的集合即可以是 List，还可以是 Map.

3. unmodifiableXxx()：返回指定集合对象的不可变视图，此处的集合即可以是List，也可以是 Set,SortedSet，还可以是 Map,SorteMap等。

   上面的三类方法的参数是原有的集合对象，返回值是该集合的 “只读” 版本。通过 Collections 提供的三类方法，可以生产只读的 Collection 或 Map。

   ```java
   public class Test {
       public static void main(String[] args) {
           //创建一个空的,不可改变的List对象
           List list=Collections.emptyList();
           //只包含一个元素的集合，且不可改变
           Set<String> set=Collections.singleton("Petterp");
   
           Map<String, Integer> map=new HashMap<>();
           map.put("1",1);
           map.put("2",1);
           //不可变的集合
           Map<String, Integer> un_map=Collections.unmodifiableMap(map);
   //        un_map.put("3",3);
       }
   }
   ```



#### Enumeration

Iterator 迭代器的古老版本

