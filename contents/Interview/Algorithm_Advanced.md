# 高级篇

## 斐波那契数列
又称兔子数列，或者黄金分割数列。指的是这样一个数列：
0、1、1、2、3、5、8、13、21……从第三项起，它的每一项都等于前两项的和。
即：F(0)=1，F(1)=1; F(n) = F(n-1) + F(n-2) (n>=2);

### 解法1
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(9));
    }

    public int fib(int n) {
        int a=0, b=1, sum = 0;
        for (int i=1; i<n; i++) {
            sum = a + b;
            a = b;
            b = sum;
        }
        return b;
    }
}
```

### 解法2
数组递归求解
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(9));
    }

    public int fib(int n) {
        int[] arr = new int[n+1];
        arr[0]=0;
        arr[1]=1;
        for(int i=2; i<=n; i++){
            arr[i] = arr[i-1]+arr[i-2];
        }
        return arr[n];
    }
}
```

### 解法3
递归求解
```
public class FibonacciDemo {

    public static void main(String[] args) {

        FibonacciDemo fibonacciDemo = new FibonacciDemo();
        System.out.println(fibonacciDemo.fib(8));
    }

    public int fib(int n) {
        if (n==0) {
            return 0;
        } else if (n==1) {
            return 1;
        }
        return fib(n-1) + fib(n-2);
    }
}
```
## 链表
### 链表反转
链表的翻转是程序员面试中出现频度最高的问题之一，常见的解决方法分为递归和迭代两种。

#### 1.非递归
```

    public Node reverse(Node head) {
        if (head==null || head.next==null) {    ////链表为空或者仅1个数直接返回
            return head;
        }
        Node newHead = null;
        Node p = head;
        while (p!=null) {
            Node tmp = p.next;          //暂存p下一个地址，防止变化指针指向后找不到后续的数
            p.next = newHead;               //p->next指向前一个空间
            newHead = p;                     //新链表的头移动到p，扩长一步链表
            p = tmp;                   //p指向原始链表p指向的下一个空间
        }
        return newHead;
    }

```

测试代码：
```
package juice.redis;

/**
 * @author Ricky Fung
 */
public class LinkListReverse {

    public static void main(String[] args) {

        LinkListReverse reverse = new LinkListReverse();
        Node head = reverse.buildLinkList();
        reverse.printLinkList(head);

        Node newHead = reverse.reverse(head);

        reverse.printLinkList(newHead);
    }

    public Node reverse(Node head) {
        if (head==null || head.next==null) {    ////链表为空或者仅1个数直接返回
            return head;
        }
        Node newHead = null;
        Node p = head;
        while (p!=null) {
            Node tmp = p.next;          //暂存p下一个地址，防止变化指针指向后找不到后续的数
            p.next = newHead;               //p->next指向前一个空间
            newHead = p;                     //新链表的头移动到p，扩长一步链表
            p = tmp;                   //p指向原始链表p指向的下一个空间
        }
        return newHead;
    }

    static class Node {
        private int value;
        private Node next;
    }

    private void printLinkList(Node head) {
        Node temp = head;
        while (temp !=null) {
            System.out.print(" " + temp.value);
            temp = temp.next;
        }
        System.out.println("");
    }

    private Node buildLinkList() {
        Node head = new Node();
        Node prev = head;
        for (int i=0; i<10; i++) {
            Node node = new Node();
            node.value = i+1;
            prev.next = node;

            prev = node;
        }
        return head;
    }
}

```

## n个整数中找出连续m个数加和是最大

## 找出一个数组中的两个数字，让这两个数字之和等于一个给定的值

## 从1亿个数中找出最大的N个数
问题：
从一亿个数中找出最大的一万个数

解题思路：
假设前一万个数是我们需要找的一万个数，但是假设不一定成立，那么，我们只需要讲后续元素与前一万个数中的最小元素比较，如果最小元素比后续元素小，则交换数据，这样只需要遍历一遍大数组就能得到正确解。时间复杂度为O(n).

## 2亿个数的文件中，统计当中出现次数最多的N个数据

## 有一个包含20亿个32位整数的大文件，在其中找到出现次数最多的数，内存限制为2GB

## 从10亿个ip找出出现次数最多的N个IP
IP可以转换为32位int，等价为 ```从10亿个整数中找出出现次数最多的N个数```。

通常比较好的方案是分治+Trie树/hash+小顶堆（就是上面提到的最小堆），即先将数据集按照Hash方法分解成多个小数据集，然后使用Trie树或者Hash统计每个小数据集中的query词频，之后用小顶堆求出每个数据集中出现频率最高的前K个数，最后在所有top K中求出最终的top K。

