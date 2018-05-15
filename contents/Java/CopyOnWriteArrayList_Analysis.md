## 概述
```java.util.concurrent.CopyOnWriteArrayList``` 用于替代同步List，在某些情况下它提供了更好的并发性能，并且在迭代期间不需要对容器进行加锁或复制。类似地，CopyOnWriteArraySet的作用是替代同步Set。

"写时复制（Copy-On-Write）"容器的线程安全性在于，只要正确地发布一个事实不可变的对象，那么在访问该对象时就不再需要进一步的同步。在每次修改时，都会创建并重新发布一个新的容器副本，从而实现可变性。

CopyOnWriteArrayList 官方doc说明如下：
> A thread-safe variant of ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array.
> 
> This is ordinarily too costly, but may be more efficient than alternatives when traversal operations vastly outnumber mutations, and is useful when you cannot or don't want to synchronize traversals, yet need to preclude interference among concurrent threads. The "snapshot" style iterator method uses a reference to the state of the array at the point that the iterator was created. This array never changes during the lifetime of the iterator, so interference is impossible and the iterator is guaranteed not to throw ConcurrentModificationException. The iterator will not reflect additions, removals, or changes to the list since the iterator was created. Element-changing operations on iterators themselves (remove, set, and add) are not supported. These methods throw UnsupportedOperationException.

> All elements are permitted, including null.
> Memory consistency effects: As with other concurrent collections, actions in a thread prior to placing an object into a CopyOnWriteArrayList happen-before actions subsequent to the access or removal of that element from the CopyOnWriteArrayList in another thread.

接下来，结合 **JDK1.7 **源码来分析CopyOnWriteArrayList内部实现机制。

## 构造方法
CopyOnWriteArrayList构造方法如下：
```
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8673264195747942595L;

    /** The lock protecting all mutators */
    transient final ReentrantLock lock = new ReentrantLock();

    /** The array, accessed only via getArray/setArray. */
    private volatile transient Object[] array;

    /**
     * Gets the array.  Non-private so as to also be accessible
     * from CopyOnWriteArraySet class.
     */
    final Object[] getArray() {
        return array;
    }

    /**
     * Sets the array.
     */
    final void setArray(Object[] a) {
        array = a;
    }

    /**
     * 无参构造函数
     */
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }

    /**
     * 带Collection构造函数
     */
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements = c.toArray();
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
        setArray(elements);
    }
}
```

CopyOnWriteArrayList内部的array数组被**volatile ** 修饰 能够保证 array数组内存可见性，当某个线程改变了 array数组，其它线程能够立刻看到，所以CopyOnWriteArrayList 所有的读操作都不用加锁，源码如下：
## 读操作

```
	/**以下皆为Read操作**/
	
    /**
     * 获取指定位置元素
     */
    public E get(int index) {
        return get(getArray(), index);
    }
	
	@SuppressWarnings("unchecked")
    private E get(Object[] a, int index) {
        return (E) a[index];
    }
	
	/**
     * 获取集合元素个数
     */
    public int size() {
        return getArray().length;
    }

    /**
     * 判断集合是否为空
     */
    public boolean isEmpty() {
        return size() == 0;
    }
	/**
     * 判断集合是否包含某个元素
     */
	public boolean contains(Object o) {
        Object[] elements = getArray();
        return indexOf(o, elements, 0, elements.length) >= 0;
    }

    /**
     * 查找某个元素位置
     */
    public int indexOf(Object o) {
        Object[] elements = getArray();
        return indexOf(o, elements, 0, elements.length);
    }
	
	public int indexOf(E e, int index) {
        Object[] elements = getArray();
        return indexOf(e, elements, index, elements.length);
    }

	private static int indexOf(Object o, Object[] elements,
                               int index, int fence) {
        if (o == null) {
            for (int i = index; i < fence; i++)
                if (elements[i] == null)
                    return i;
        } else {
            for (int i = index; i < fence; i++)
                if (o.equals(elements[i]))
                    return i;
        }
        return -1;
    }
	
    public int lastIndexOf(Object o) {
        Object[] elements = getArray();
        return lastIndexOf(o, elements, elements.length - 1);
    }

    public int lastIndexOf(E e, int index) {
        Object[] elements = getArray();
        return lastIndexOf(e, elements, index);
    }

    private static int lastIndexOf(Object o, Object[] elements, int index) {
        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elements[i] == null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elements[i]))
                    return i;
        }
        return -1;
    }
```

** volatile** 可以保证 可见性，但无法保证原子性，CopyOnWriteArrayList 所有的修改操作都需要通过 ReentrantLock 加锁来保证任意时刻只有一个线程能够修改CopyOnWriteArrayList 。
关于 ReentrantLock 内部实现可以参考我的另外一篇文章：
[JUC系列 - ReentrantLock源码解析](http://www.jianshu.com/p/eac98e598053).

## 更新操作
CopyOnWriteArrayList 几个重要修改操作的源代码如下：
```
	/**以下皆为Write操作**/
	
	/**
     * 替换指定位置上的元素
     */
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 向集合添加元素
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 向集合指定位置添加元素
     */
    public void add(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            if (index > len || index < 0)
                throw new IndexOutOfBoundsException("Index: "+index+
                                                    ", Size: "+len);
            Object[] newElements;
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(elements, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index, newElements, index + 1,
                                 numMoved);
            }
            newElements[index] = element;
            setArray(newElements);
        } finally {
            lock.unlock();
        }
    }

    /**
     * 删除指定元素
	 * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 删除元素
     */
    public boolean remove(Object o) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            if (len != 0) {
                // Copy while searching for element to remove
                // This wins in the normal case of element being present
                int newlen = len - 1;
                Object[] newElements = new Object[newlen];

                for (int i = 0; i < newlen; ++i) {
                    if (eq(o, elements[i])) {
                        // found one;  copy remaining and exit
                        for (int k = i + 1; k < len; ++k)
                            newElements[k-1] = elements[k];
                        setArray(newElements);
                        return true;
                    } else
                        newElements[i] = elements[i];
                }

                // special handling for last cell
                if (eq(o, elements[newlen])) {
                    setArray(newElements);
                    return true;
                }
            }
            return false;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 删除指定范围段所有元素
     * @throws IndexOutOfBoundsException if fromIndex or toIndex out of range
     *         ({@code{fromIndex < 0 || toIndex > size() || toIndex < fromIndex})
     */
    private void removeRange(int fromIndex, int toIndex) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;

            if (fromIndex < 0 || toIndex > len || toIndex < fromIndex)
                throw new IndexOutOfBoundsException();
            int newlen = len - (toIndex - fromIndex);
            int numMoved = len - toIndex;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, newlen));
            else {
                Object[] newElements = new Object[newlen];
                System.arraycopy(elements, 0, newElements, 0, fromIndex);
                System.arraycopy(elements, toIndex, newElements,
                                 fromIndex, numMoved);
                setArray(newElements);
            }
        } finally {
            lock.unlock();
        }
    }

    /**
     * 
     */
    public boolean addIfAbsent(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // Copy while checking if already present.
            // This wins in the most common case where it is not present
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = new Object[len + 1];
            for (int i = 0; i < len; ++i) {
                if (eq(e, elements[i]))
                    return false; // exit, throwing away copy
                else
                    newElements[i] = elements[i];
            }
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

可以看到，所有修改操作之前都必须先通过 ```lock.lock();```来加锁，修改完成后 通过 ```setArray(newElements);``` 来发布新的array数组，最后通过 ```lock.unlock();```释放锁。

### 应用场景
显然，每次修改容器时都会复制底层数组，这需要一定的开销，特别是当容器的规模较大时。仅当迭代操作远远多于修改操作时，才应该使用"写时复制（Copy-On-Write）"容器。例如事件通知系统：在分发通知时需要迭代已注册监听器链表，并调用每一个监听器，在大多数情况下，注册和注销事件监听器的操作远远少于接收事件通知的操作。
