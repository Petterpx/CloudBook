# Java | 关于synchronized相关理解

## 背景

**资源冲突**

Java本身是支持多线程的，而在多线程的情况下，为了防止 `多个任务同时访问同一个资源而导致的冲突问题`，所以出现了加锁机制。也就是说第一个访问某项资源的任务必须锁定这项资源，使其他任务在其被解锁之前，就无法访问它，而在其被解锁时候，另一个任务就可以锁定并使用它。

所以Java提供了关键字 `synchronized` ,为防止资源冲突。当任务希望执行被`synchronized` 关键字保护的代码片段时，`Java`  编译器会生成代码已查看锁是否可用。如果可用，该任务获取锁，执行代码，然后释放锁。

## 对象锁&&方法锁

所有对象都自动包含 **独立的锁** ，当调用对象上任何 `synchronized` 方法，此对象将被加锁，并且该对象上的其他 `synchronized` 方法调用只有等到前一个方法执行完成并释放了锁之后才能被调用。

对于 **方法锁** ，其实和对象锁差别不大，方法锁针对的是一个方法，而对象锁则是针对于某一个对象的实例，从而锁定的是对应的一个代码块。

### 修饰多个方法

![image-20201128180923809](https://tva1.sinaimg.cn/large/0081Kckwly1gl51sgob5uj30nr0gln30.jpg)

**结论：** 线程按序执行

---

### 修饰多个代码块

![image-20201128182155537](https://tva1.sinaimg.cn/large/0081Kckwly1gl525ih1aoj30jc0ehju2.jpg)

**结论:**  按调用顺序依次执行

---

### 修饰一个方法，另一个不修饰

![image-20201128182750402](https://tva1.sinaimg.cn/large/0081Kckwly1gl52bnw4f7j30je0el0vf.jpg)

**结论：** 线程交替执行

---

### 修饰一个对象，对象中的方法都不加锁

![image-20201128183628604](https://tva1.sinaimg.cn/large/0081Kckwly1gl52knatutj30jm0d8acd.jpg)

**结论： **线程交替执行

---

### 修饰一个对象，对象中的方法都加锁

![image-20201128183957378](https://tva1.sinaimg.cn/large/0081Kckwly1gl52o9n1ohj30ia0dago7.jpg)

**结论：**  线程按序执行

---

### 修饰一个对象，对象中的方法一个加锁另一个不加锁

![image-20201128194547847](https://tva1.sinaimg.cn/large/0081Kckwly1gl54krquugj30ht0bn0vl.jpg)

**结论：**  线程交替执行

---

## 类锁

**synchronized 修饰的方法或代码块**

由于一个class 中的静态方法及静态变量在内存中只有一份，无论这个class类被实例化多少次。所以，一旦一个 静态方法被声明为 synchronized ，此类所有实例化对象在调用此方法时，都共用一把锁，所以称之为类锁。类锁常用于控制静态方法之间的同步。

### 修饰一个类

![image-20201128184408715](https://tva1.sinaimg.cn/large/0081Kckwly1gl52smbsjoj30ih0cmtbc.jpg)

**结论：** 线程交替执行

---

### 同时执行类中修饰过的静态方法和没修饰过的

![image-20201128185905133](https://tva1.sinaimg.cn/large/0081Kckwly1gl5386gvf0j30ix0ct40r.jpg)

**结果：**线程交替执行

---

### 同时执行类中修饰过的静态方法

![image-20201128190649044](https://tva1.sinaimg.cn/large/0081Kckwly1gl53g7ur9mj30j00ekjue.jpg)

**结论：**  线程按序执行





## 参考：

- Java编程思想-并发底层原理
- [synchronized修饰不同位置的作用](https://blog.csdn.net/u013041642/article/details/88933136)
- [方法锁、对象锁、类锁的意义和区别](https://blog.csdn.net/baidu_40389775/article/details/87533150?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160654843719195283043862%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=160654843719195283043862&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v1~rank_blog_v1-1-87533150.pc_v1_rank_blog_v1&utm_term=%E9%94%81&spm=1018.2118.3001.4450)