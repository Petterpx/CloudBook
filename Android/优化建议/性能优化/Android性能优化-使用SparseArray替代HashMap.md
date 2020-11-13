## 什么是稀疏数组？

所谓稀疏数组就是数组中大部分的内容值都未被使用(或都为0)，在数组中仅有少部分的空间使用。因此造成内存空间的浪费，为了节省内存空间，并且不影响数组中原有的内容值，我们可以采用一种压缩的方式来表示稀疏数组的内容。





## SparseArray

SparseArray比HashMap更省内存，在某些条件下性能更好，主要是由于它避免了对key的自己主动装箱（int转为Integer类型），它内部则是通过两个数组来进行数据存储的。一个存储key，另外一个存储value，为了优化性能，它内部对数据还採取了压缩的方式来表示稀疏数组的数据，从而节约内存空间。我们从源代码中能够看到key和value各自是用数组表示：

```
    private int[] mKeys;
    private Object[] mValues;
```

我们能够看到，SparseArray仅仅能存储key为int类型的数据。同一时候，SparseArray在存储和读取数据时候，使用的是二分查找法，我们能够看看：

```
 public void put(int key, E value) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        ...
        }
 public E get(int key, E valueIfKeyNotFound) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        ...
        }
```

也就是在put加入数据的时候。会使用二分查找法和之前的key比較当前我们加入的元素的key的大小，然后依照从小到大的顺序排列好，所以，SparseArray存储的元素都是按元素的key值从小到大排列好的。 
而在获取数据的时候，也是使用二分查找法推断元素的位置，所以，在获取数据的时候非常快，比HashMap快的多，由于HashMap获取数据是通过遍历Entry[]数组来得到相应的元素。



### 加入数据

```
public void put(int key, E value)
```

### 删除数据

```
 public void remove(int key)
```

or

```
public void delete(int key)
```

事实上remove内部还是通过调用delete来删除数据的

### 获取数据

```
public E get(int key)
```

or

```
public E get(int key, E valueIfKeyNotFound)
```

该方法可设置假设key不存在的情况下默认返回的value

### 特有方法

在此之外，SparseArray还提供了两个特有方法。更方便数据的查询： 
获取相应的key：

```
public int keyAt(int index)
```

获取相应的value：

```
public E valueAt(int index)
```

### **SparseArray应用场景：**

虽说SparseArray性能比較好，可是由于其加入、查找、删除数据都须要先进行一次二分查找。所以在数据量大的情况下性能并不明显，将减少至少50%。

满足下面两个条件我们能够使用SparseArray取代HashMap：

- 数据量不大，最好在千级以内
- key必须为int类型，这中情况下的HashMap能够用SparseArray取代：

```
HashMap<Integer, Object> map = new HashMap<>();
用SparseArray取代:
SparseArray<Object> array = new SparseArray<>();
```

## **ArrayMap**

这个api的资料在网上能够说差点儿没有，然并卵，仅仅能看文档了 
ArrayMap是一个<**key,value**>映射的数据结构，它设计上很多其他的是考虑内存的优化，内部是使用两个数组进行数据存储，一个数组记录key的hash值。另外一个数组记录Value值。它和SparseArray一样。也会对key使用二分法进行从小到大排序，在加入、删除、查找数据的时候都是先使用二分查找法得到相应的index，然后通过index来进行加入、查找、删除等操作，所以，应用场景和SparseArray的一样，假设在数据量比較大的情况下，那么它的性能将退化至少50%。

### 加入数据

```
public V put(K key, V value)
```

### 获取数据

```
public V get(Object key)
```

### 删除数据

```
public V remove(Object key)
```

### 特有方法

它和SparseArray一样相同也有两个更方便的获取数据方法：

```
public K keyAt(int index)
public V valueAt(int index)
```

### **ArrayMap应用场景**

- 数据量不大，最好在千级以内
- 数据结构类型为Map类型

```
ArrayMap<Key, Value> arrayMap = new ArrayMap<>();
```

【注】：假设我们要兼容aip19下面版本号的话，那么导入的包须要为v4包

```
import android.support.v4.util.ArrayMap;
```

## **总结**

SparseArray和ArrayMap都差点儿相同，使用哪个呢？ 
假设数据量都在千级以内的情况下：

> 1、假设key的类型已经确定为int类型。那么使用SparseArray，由于它避免了自己主动装箱的过程，假设key为long类型，它还提供了一个LongSparseArray来确保key为long类型时的使用
>
> 2、假设key类型为其他的类型，则使用ArrayMap