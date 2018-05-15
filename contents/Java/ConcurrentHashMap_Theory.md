## 前言
以前写过介绍HashMap的文章，文中提到过HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

我们来了解另一个键值存储集合HashTable，它是线程安全的，它在所有涉及到多线程操作的都加上了synchronized关键字来锁住整个table，这就意味着所有的线程都在竞争一把锁，在多线程的环境下，它是安全的，但是无疑是效率低下的。

其实HashTable有很多的优化空间，锁住整个table这么粗暴的方法可以变相的柔和点，比如在多线程的环境下，对不同的数据集进行操作时其实根本就不需要去竞争一个锁，因为他们不同hash值，不会因为rehash造成线程不安全，所以互不影响，这就是锁分离技术，将锁的粒度降低，利用多个锁来控制多个小的table，这就是这篇文章的主角ConcurrentHashMap JDK1.7版本的核心思想。

## JDK 1.7实现
在JDK1.7版本中，ConcurrentHashMap的数据结构是由一个Segment数组和多个HashEntry组成，如下图所示：
![](https://github.com/TFdream/blog/blob/master/docs/image/JDK_Source/concurrenthashmap_jdk7.png)

Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是上面的提到的锁分离技术，而每一个Segment元素存储的是HashEntry数组+链表，这个和HashMap的数据存储结构一样。

### 初始化
ConcurrentHashMap的初始化是会通过位与运算来初始化Segment的大小，用ssize来表示，如下所示：
```

```

### put操作

### size实现


## JDK 1.8实现
JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，结构如下：

![](https://github.com/TFdream/blog/blob/master/docs/image/JDK_Source/concurrenthashmap_jdk8.png)

### 初始化

### put操作

### size实现

## 参考资料
[谈谈ConcurrentHashMap1.7和1.8的不同实现](http://www.importnew.com/23610.html)

