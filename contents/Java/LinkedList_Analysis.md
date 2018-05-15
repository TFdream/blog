## 数据结构
LinkedList底层的数据结构是基于双向循环链表的，如果对双向链表这个数据结构很熟悉的话，学习 LinkedList 就没什么难度了。下面是双向链表的结构：

![](https://github.com/TFdream/blog/blob/master/docs/image/JDK_Source/LinkedList_jdk8.png)

## 源码分析
本文基于 **JDK 1.8** 源码进行分析。

### 构造方法
LinkedList 构造方法如下：
```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;

    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }
}
```
特性如下：
* LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
* LinkedList 实现 List 接口，能对它进行队列操作。
* LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
* LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
* LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。

### add操作
add()方法如下：
```
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
	
	/**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

```

因为是链表结构，所以添加时无须扩容，只修改size。

### get操作
```
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
	
	/**
     * Returns the (non-null) Node at the specified element index.
     */
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
	
	private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
	/**
     * Tells if the argument is the index of an existing element.
     */
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
```
首先调用checkElementIndex判断index是否合法（index >= 0 && index < size），然后根据 index是否大于当前size的一半决定是从头结点开始往后查询，还是从尾节点开始往前查询。

