
# JVM

## 一、JVM运行时数据区
Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，已经创建和销毁时间，有的区域随着虚拟机进程的启动而创建，有些区域则依赖用户线程的启动和结束而创建和销毁。根据《Java虚拟机规范（Java SE 7）》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域，如下图所示：
![Jvm_Area](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_Area.jpg)

### 堆（Heap）
Java堆分代如下：
![Jvm_Heap](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_heap_area.png)

#### 1. 新生代（Young Generation）
新生代是所有新对象产生的地方。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。年轻代分为3个部分：Enden区和Survivor From和Survivor To区。
新生代的主要特点：
* 大多数新建的对象都位于Eden区。
* 当Eden区被对象填满时，就会执行Minor GC。并把所有存活下来的对象转移到其中一个Survivor 区。
* Minor GC同样会检查存活下来的对象，并把它们转移到另一个Survivor 区。这样在一段时间内，总会有一个空的Survivor 区。
“熬过”多次Minor GC后，仍然存活下来的对象会被转移到年老代内存空间。如果对象在Eden出生并经过第一次Minor GC后任然存活，并且能被Survivor 容纳的话，它将被移动到Survivor 空间中，并且对象年龄设为1。对象在Survivor 区中每熬过”一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就将会被晋升到老年代中。

年轻代大小可以通过参数 -Xmn10M来控制；Eden区和Survivor区的大小可以通过参数 -XX:SurvivorRatio来进行控制，默认为8：1。

### 年老代（Old Generation）
年老代包含了长期存活的对象和经过多次Minor GC后依然存活下来的对象。通常会在老年代内存被占满时进行垃圾回收。老年代的垃圾收集叫做Major GC，Major GC会花费更多的时间。

Minor GC与Major GC的区别
* 新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
* 老年代GC（Major GC/Full GC）：指发生在老年代的垃圾收集工作，出现了Major GC，经常会伴随至少一次的Minor GC（但并非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程），Major GC的速度一般会比Minor GC慢10倍以上。

## 二、内存分配与回收策略
Java技术体系中所提倡的自动内存管理最终可归结为自动化的解决了两个问题：给对象分配内存以及回收分配给对象的内存。

对象的内存分配，往大方向上讲，就是在堆上分配（但也可能经过JIT编译后被拆散为标量类型并间接地栈上分配），对象主要分配在新生代的Eden区上，如果启动了本地线程分配缓冲，将按线程优先在TLAB上分配。

接下来会讲解几条最普遍的内存分配策略，并通过代码去验证这些规则。
### 1、对象优先在Eden分配
大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。

测试代码：
```
public class App {
    public static final int _1MB = 1024*1024;

    /**
     * VM参数：-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
     */
    public static void testAllocation(){

        byte[] allocation1, allocation2, allocation3, allocation4;
        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[2 * _1MB];
        allocation3 = new byte[2 * _1MB];
        allocation4 = new byte[2 * _1MB];
    }

    public static void main( String[] args ) {

        testAllocation();
    }
}
```

### 2. 大对象直接进入老年代
所谓的大对象是指，需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串已经数组（上例中的byte[] 就是典型的大对象）。

大对象对虚拟机的内存分配来说就是一个坏消息（比遇到一个大对象更加坏的消息就是遇到一群 “朝生夕灭”的 "短命大对象"，写程序的时候应当避免），经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来 "安置" 它们。

### 3. 长期存活的对象将进入老年代
既然虚拟机采用分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了能做到这点，虚拟机给每个对象定义了一个对象年龄(Age) 计数器。如果对象在Eden出生并经过第一次Minor GC后任然存活，并且能被Survivor 容纳的话，它将被移动到Survivor 空间中，并且对象年龄设为1。对象在Survivor 区中每熬过”一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就将会被晋升到老年代中。对象晋升老年代的 年龄阈值，可以通过参数 -XX:MaxTenuringThreshold 设置。

### 4. 动态对象年龄判定
为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold 才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或者等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold 中要求的年龄。


## 三、JVM垃圾回收算法
Java虚拟机垃圾回收(二) 垃圾回收算法：https://blog.csdn.net/tjiyu/article/details/53983064


## 4.JVM垃圾收集器
Java虚拟机垃圾回收(三) 7种垃圾收集器：https://blog.csdn.net/tjiyu/article/details/53983650


## 参考资料

