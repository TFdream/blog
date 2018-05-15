## ThreadLocal概述
ThreadLocal提供了线程本地变量，它可以保证访问到的变量属于当前线程，每个线程都保存有一个变量副本，每个线程的变量都不同。ThreadLocal相当于提供了一种线程隔离，将变量与线程相绑定。

> This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).
> 
> Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).

## ThreadLocal源码分析

### ThreadLocal 构造方法
构造方法如下：
```
public class ThreadLocal<T> {

	private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
        new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
	
    public ThreadLocal() {
    }
}
```
ThreadLocal通过threadLocalHashCode来标识每一个ThreadLocal的唯一性。threadLocalHashCode通过CAS操作进行更新，每次hash操作的增量为 0x61c88647（不知为何）。

### set方法
接下来，看看它的set方法：
```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

通过Thread.currentThread()方法获取了当前的线程引用，并传给了getMap(Thread)方法获取一个ThreadLocalMap的实例。我们继续跟进getMap(Thread)方法：
```
 ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

可以看到getMap(Thread)方法直接返回Thread实例的成员变量threadLocals。它的定义在Thread内部，访问级别为package级别：
```

public class Thread implements Runnable {
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
	
}
```

到了这里，我们可以看出，每个Thread里面都有一个ThreadLocal.ThreadLocalMap成员变量，也就是说每个线程通过ThreadLocal.ThreadLocalMap与ThreadLocal相绑定，这样可以确保每个线程访问到的thread-local variable都是本线程的。

我们往下继续分析。获取了ThreadLocalMap实例以后，如果它不为空则调用ThreadLocalMap.ThreadLocalMap 的set方法设值；若为空则调用ThreadLocal 的createMap方法new一个ThreadLocalMap实例并赋给Thread.threadLocals。

ThreadLocal 的 createMap方法的源码如下：
```
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

### get方法
ThreadLocal 的 get 方法，源码如下：
```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }
```

通过Thread.currentThread()方法获取了当前的线程引用，并传给了getMap(Thread)方法获取一个ThreadLocalMap的实例。继续跟进setInitialValue()方法：
```
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

首先调用 initialValue()方法来初始化，然后 通过Thread.currentThread()方法获取了当前的线程引用，并传给了getMap(Thread)方法获取一个ThreadLocalMap的实例，并将 初始化值存到ThreadLocalMap 中。

initialValue() 源码如下：
```
  protected T initialValue() {
        return null;
    }
```

下面我们探究一下ThreadLocalMap的实现。

### ThreadLocalMap
ThreadLocalMap是ThreadLocal的静态内部类，源码如下：
```
public class ThreadLocal<T> {

	static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The initial capacity -- MUST be a power of two.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;

        /**
         * The number of entries in the table.
         */
        private int size = 0;

        /**
         * The next size value at which to resize.
         */
        private int threshold; // Default to 0

        ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
	}
	
}
```

其中INITIAL_CAPACITY代表这个Map的初始容量；1是一个Entry类型的数组，用于存储数据；size代表表中的存储数目；threshold代表需要扩容时对应size的阈值。

**ThreadLocalMap key为当前ThreadLocal对象，value为我们要设置的对象**。

Entry类是ThreadLocalMap的静态内部类，用于存储数据。

Entry类继承了WeakReference<ThreadLocal<?>>，即每个Entry对象都有一个ThreadLocal的弱引用（作为key），这是为了防止内存泄露。一旦线程结束，key变为一个不可达的对象，这个Entry就可以被GC了。

接下来我们来看ThreadLocalMap 的set方法的实现：
```
        private void set(ThreadLocal key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

ThreadLocal 的get方法会调用 ThreadLocalMap 的 getEntry(ThreadLocal key) ，其源码如下：
```
	private Entry getEntry(ThreadLocal key) {
		int i = key.threadLocalHashCode & (table.length - 1);
		Entry e = table[i];
		if (e != null && e.get() == key)
			return e;
		else
			return getEntryAfterMiss(key, i, e);
	}

	private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
		Entry[] tab = table;
		int len = tab.length;

		while (e != null) {
			ThreadLocal k = e.get();
			if (k == key)
				return e;
			if (k == null)
				expungeStaleEntry(i);
			else
				i = nextIndex(i, len);
			e = tab[i];
		}
		return null;
	}
```

### 参考资料
[并发编程 | ThreadLocal源码深入分析](http://www.sczyh30.com/posts/Java/java-concurrent-threadlocal/)
