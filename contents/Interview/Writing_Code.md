
## 编程实现一个简单的HashMap
```
package io.mindflow.elasticjob;

/**
 * @author Ricky Fung
 */
public class HashMap<K, V> {

    //默认初始化化容量,即16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

    //最大容量，即2的30次方
    static final int MAXIMUM_CAPACITY = 1 << 30;

    //默认装载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    //HashMap内部的存储结构是一个数组，此处数组为空，即没有初始化之前的状态
    static final Entry<?,?>[] EMPTY_TABLE = {};

    //空的存储实体
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

    //实际存储的key-value键值对的个数
    transient int size;

    //阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold
    int threshold;

    //负载因子，代表了table的填充度有多少，默认是0.75
    final float loadFactor;

    //用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
    transient int modCount;

    //通过初始容量和状态因子构造HashMap
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)//参数有效性检查
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)//参数有效性检查
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))//参数有效性检查
            throw new IllegalArgumentException("Illegal load factor: " +
                    loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
    }

    //通过扩容因子构造HashMap,容量去默认值，即16
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    //装载因子取0.75，容量取16，构造HashMap
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    static class Entry<K, V> {
        final K key;
        V value;
        Entry<K,V> next;
        final int hash;

        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public K getKey() {
            return key;
        }

        public V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public V getValue() {
            return value;
        }
    }

    public V put(K key, V value) {
        //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4(=16)
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);//分配数组空间
        }
        //如果key为null，存储位置为table[0]或table[0]的冲突链上
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);//对key的hashcode进一步计算，确保散列均匀
        int i = indexFor(hash, table.length);//获取在table中的实际位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);//调用value的回调函数，其实这个函数也为空实现
                return oldValue;
            }
        }
        modCount++;//保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        addEntry(hash, key, value, i);//新增一个entry
        return null;
    }

    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容，新容量为旧容量的2倍
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);//扩容后重新计算插入的位置下标
        }

        //把元素放入HashMap的桶的对应位置
        createEntry(hash, key, value, bucketIndex);
    }

    //创建元素
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];  //获取待插入位置元素
        table[bucketIndex] = new Entry<>(hash, key, value, e);//这里执行链接操作，使得新插入的元素指向原有元素。
        //这保证了新插入的元素总是在链表的头
        size++;//元素个数+1
    }
    
    //按新的容量扩容Hash表
    void resize(int newCapacity) {
        Entry[] oldTable = table;//老的数据
        int oldCapacity = oldTable.length;//获取老的容量值
        if (oldCapacity == MAXIMUM_CAPACITY) {//老的容量值已经到了最大容量值
            threshold = Integer.MAX_VALUE;//修改扩容阀值
            return;
        }
        //新的结构
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));//将老的表中的数据拷贝到新的结构中
        table = newTable;//修改HashMap的底层数组
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);//修改阀值
    }
    //返回数组下标
    static int indexFor(int h, int length) {
        return h & (length-1);
    }

    //用了很多的异或，移位等运算，对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
    final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {//这里针对String优化了Hash函数，是否使用新的Hash函数和Hash因子有关
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    private void inflateTable(int toSize) {
        int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);//此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
        table = new Entry[capacity];//分配空间
        initHashSeedAsNeeded(capacity);//选择合适的Hash因子
    }

    private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }
}

```

参考：[HashMap源码解析（基于JDK1.7）](https://blog.csdn.net/xiaokang123456kao/article/details/77503784)

## 编程实现一个延时队列
时间轮（TimingWheel）

Kafka时间轮的实现是TimingWheel，他是一个存储定时任务的环形队列（桶），底层使用数组实现，数组中每一个元素可以存放一个TimerTaskList对象
TimerTaskList是环形双向链表，在其中链表项TimeTaskEntry封装了真正的定时任务TimerTask。TimerTaskList使用expiration字段记录了整个TimerTaskList的超时时间。TimeTaskEntry中的expirationMs字段记录了超时时间戳，timerTask字段指向了对应的TimerTask任务.
