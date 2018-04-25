
# JVM

## 一、JVM运行时数据区
Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，已经创建和销毁时间，有的区域随着虚拟机进程的启动而创建，有些区域则依赖用户线程的启动和结束而创建和销毁。根据《Java虚拟机规范（Java SE 7）》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域，如下图所示：

![Jvm_Area](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_Area.jpg)

### 1. 程序计数器
程序计数器（Program Counter Register）是一块较小的内存空间，它可以看做是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令、分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的。在任何一个确定的时刻，一个处理器都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各个线程之间计数器互不影响，独立存储。

如果线程正在执行的是一个Java方法，那这个计数器记录的是正在执行的字节码指令的地址；如果正在执行的是Native方法，这个计数器值则为空（undefined）。 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

程序计数器是线程私有的，它的生命周期与线程相同（随线程而生，随线程而灭）。

### 2. Java虚拟机栈
虚拟机栈（Java Virtual Machine Stack）描述的是Java方法执行的内存模型：每个方法被执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从被调用直至执行完成的过程就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

在Java虚拟机规范中，对这个区域规定了两种异常情况：
* 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；
* 如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可以扩展），如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

与程序寄存器一样，java虚拟机栈也是线程私有的，它的生命周期与线程相同。

### 3. 本地方法栈
本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用是非常类似，它们之间的区别在于虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则是为虚拟机使用到的Native方法服务。在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构 并没有强制规定，因此具体的虚拟机可以自由的实现它。

与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

与虚拟机栈一样，本地方法栈也是线程私有的。

### 4. Java 堆（Java Heap）
对于大多数应用来说，Java 堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动的是创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都要在这里分配内存。

Java堆是垃圾收集器管理的主要区域，因此很多时候也被称为“GC堆”（Garbage Collected Heap）。从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆还可以细分为：新生代和老年代；新生代又可以分为：Eden 空间、From Survivor空间、To Survivor空间。

根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的（通过-Xms和-Xmx控制）。如果在堆中没有内存完成实例的分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

### 5. 方法区（Method Area）
方法区（Method Area）和Java堆一样，是各个线程共享的内存区域，它用于存放已被虚拟机加载的类信息、常量、静态变量、JIT编译后的代码等数据。方法区在虚拟机启动的时候创建。

对于习惯在Hotspot虚拟机上开发、部署应用程序的开发者来说，很多人都更愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为Hotspot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已，这样Hotspot的垃圾收集器可用像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。对于其他虚拟机（如BEA JRockit、IBM J9等）来说，是不存在永久代的概念的。

Java虚拟机规范对方法区的限制非常宽松，除了和堆一样不需要不连续的内存空间和可以固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样"永久"存在了。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说，这个区域的回收"成绩" 比较难以让人满意。

根据Java虚拟机规范的规定，如果方法区的内存空间不能满足内存分配需要时，将抛出OutOfMemoryError异常。

### 6. 运行时常量池
运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

### 7. 直接内存
直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但是这部分内存也被频繁使用，而且也可能导致OutOfMemoryError异常出现。

在JDK 1.4 中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方法，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

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


## 四、JVM垃圾收集器
先来了解HotSpot虚拟机中的7种垃圾收集器：Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS、G1，先介绍一些垃圾收集的相关概念，再介绍它们的主要特点、应用场景、以及一些设置参数和基本运行原理。

### 1. 垃圾收集器概述
垃圾收集器是垃圾回收算法（标记-清除算法、复制算法、标记-整理算法、火车算法）的具体实现，不同商家、不同版本的JVM所提供的垃圾收集器可能会有很在差别，本文主要介绍HotSpot虚拟机中的垃圾收集器。

### 2. 垃圾收集器组合
JDK7/8后，HotSpot虚拟机所有收集器及组合（连线），如下图：
![](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_collector.png)

#### A. 图中展示了7种不同分代的收集器：
Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS、G1；

#### B. 它们所处区域，则表明其是属于新生代收集器还是老年代收集器：

* 新生代收集器：Serial、ParNew、Parallel Scavenge；
* 老年代收集器：Serial Old、Parallel Old、CMS；
* 整堆收集器：G1；

#### C. 两个收集器间有连线，表明它们可以搭配使用：

Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1；

其中Serial Old作为CMS出现"Concurrent Mode Failure"失败的后备预案（后面介绍）；


Java虚拟机垃圾回收(三) 7种垃圾收集器：https://blog.csdn.net/tjiyu/article/details/53983650


## 参考资料

