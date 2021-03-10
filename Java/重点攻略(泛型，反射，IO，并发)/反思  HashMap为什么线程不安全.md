# 反思 | HashMap为什么线程不安全



有一天，当我想要写篇关于 ConcurrentHashMap 相关时，下意识就使用HashMap去和它对比，自然就在电脑上输入了以下：

![image-20201129150825237](https://tva1.sinaimg.cn/large/0081Kckwly1gl626hlbvuj30aw048jrc.jpg)

`HashMap是线程不安全的` ，为什么?

当思绪在我脑海里飞转一圈后，我发现，自己也并不完全解释清楚为什么HashMap是线程不安全。然而我自己并不明白却还要假装明白，并以一种本来即如此，特别easy的风格打出上面这句话。想到此处瞬间如鲠在喉，羞愧难安，为什么这种问题常常会出现在身上。

反思过往，始终认为，当我将技术文字映然纸上，面对大众时，就应该对其所言负责，也是对自己的严谨。本篇即也就是我对自己知识体系的一个补充，也希望对你有所帮助。





对于HashMap，我们可能不能再熟悉了，在jdk1.7版本，已数组加链表的方式存储数据，Jdk1.8新增了链表转红黑树，具体的概念并不属于本篇该探讨范围。

为什么在多线程情况下，会出现线程不安全呢？



## 源码是最好的老师

HashMap#putVal

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
  	//初始化数组
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
  	//节点不存在，直接存储为链表
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
      	//如果节点已经存在
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
      	//如果已经转为红黑树了
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
      	//否则当前就是链表
        else {
          	//排序
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
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
  	//
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```





