
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

从图中可以看出： 堆大小 = 新生代 + 老年代。其中，堆的大小可以通过参数 –Xms、-Xmx 来指定。
默认的，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定)，即：新生代 ( Young ) = 1/3 的堆空间大小,老年代 ( Old ) = 2/3 的堆空间大小。

其中，新生代 ( Young )被细分为 Eden 和 两个 Survivor 区域，这两个 Survivor 区域分别被命名为 from 和 to，以示区分。
默认的，Edem : from : to = 8 :1 : 1 ( 可以通过参数–XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。

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

### 分配担保
如果另一块Survivor空间没有足够空间存放上一次新生代收集下来的存活对象时，这些对象将直接通过分配担保机制（Handle Promotion）进入老年代；

## 三、JVM垃圾回收算法
说到垃圾收集（Garbage Collection，GC），大部分人都会把这项技术当做 Java 语言的伴生产物。事实上，GC的历史比Java久远，1960年诞生于MIT的Lisp是第一门真正使用内存动态分配和垃圾收集技术的语言。当Lisp还在胚胎时期时，人们就在思考GC需要完成的3件事情：
* 哪些内存需要回收？
* 什么时候回收？
* 如何回收？

### 如何判定对象已死亡
#### 对象“存活”判定算法
在堆里面存放着Java世界中几乎所有的对象实例，垃圾收集器在对堆进行回收前，第一件事情就是要确定这些对象之中哪些还“存活”着，哪些已经“死亡”（即不可能被任何途径使用的对象）。

#### 1. 引用计数算法
原理：给对象中添加一个引用计数器，每当有一个地方引用它时，计数器加1；引用失效时，计数器减1；任何时刻计数器为0的对象就是不可能再被使用的。 

缺点：很难解决对象相互循环引用的问题（两个对象相互循环引用，但其实他们都已经没有用了）。

#### 2. 可达性分析算法
在主流的商用程序语言（Java、C#、Lisp）的主流实现中都是通过可达性分析（Reachability Analysis）来判定对象是否存活的。

原理：通过一些列称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的。
在Java语言中，可作为GC Roots的对象包括下面几种：
* 虚拟机栈（栈帧中的本地变量表）中引用的对象。
* 方法区中类静态属性引用的对象。
* 方法区中常量引用的对象。
* 本地方法栈中JNI（即一般说的Native方法）引用的对象。

![Jvm_GC_Root](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_GC_Root.png)


### 引用
无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象的引用链是否可达，判定对象是否存活都与“引用”有关。

在JDK1.2之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）4种，这4种应用强度依次逐渐减弱。

#### 强引用
强引用就是指在程序代码之中普遍存在的，类似“Object object = new Object()
”这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。

#### 软引用
软引用是用来描述一些还在用但并非必需的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收，如果这次回收完成还没有足够的内存，才会抛出内存溢出异常。在JDK1.2之后，提供了SoftReference类来实现软引用。

#### 弱引用
弱引用也是用来描述非必需对象的，但是它的强度比软引用要更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK1.2之后，提供了WeakReference类来实现弱引用。

#### 虚引用
虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。在JDK1.2之后，提供了PhantomReference类来实现虚引用。


### 垃圾回收算法概述
先来了解Java虚拟机垃圾回收的几种常见算法：标记-清除算法、复制算法、标记-整理算法、分代收集算法、火车算法，介绍它们的算法思路，有什么优点和缺点，以及主要应用场景。

### 1、标记-清除算法
标记-清除（Mark-Sweep）算法是一种基础的收集算法。
#### 算法思路
1. 标记：首先标记出所有需要回收的对象；
    - 第一次标记：在可达性分析后发现对象到GC Roots没有任何引用链相连时，被第一次标记；
    - 第二次标记：GC将对F-Queue队列中的对象进行第二次小规模标记；
2. 清除：两次标记后，还在"即将回收"集合的对象将被统一回收；

#### 优点
基于最基础的可达性分析算法，它是最基础的收集算法；
而后续的收集算法都是基于这种思路并对其不足进行改进得到的；

#### 缺点
主要有两个缺点：
* 效率问题：标记和清除两个过程的效率都不高；
* 空间问题
    - 标记清除后会产生大量不连续的内存碎片；
    - 这会导致分配大内存对象时，无法找到足够的连续内存，从而需要提前触发另一次垃圾收集动作；

#### 应用场景
针对老年代的CMS收集器；

### 2、复制算法算法
"复制"（Copying）收集算法，为了解决标记-清除算法的效率问题；

#### 算法思路
1. 把内存划分为大小相等的两块，每次只使用其中一块；
2. 当一块内存用完了，就将还存活的对象复制到另一块上（而后使用这一块）；
3. 再把已使用过的那块内存空间一次清理掉，而后重复步骤2；   

#### 优点
* 这使得每次都是只对整个半区进行内存回收；
* 内存分配时也不用考虑内存碎片等问题（可使用"指针碰撞"的方式分配内存）；
* 实现简单，运行高效；
      
#### 缺点
* 空间浪费：可用内存缩减为原来的一半，太过浪费（解决：可以改良，不按1:1比例划分）；
* 效率随对象存活率升高而变低：当对象存活率较高时，需要进行较多复制操作，效率将会变低（解决：后面的标记-整理算法）；

#### 应用场景
现在商业JVM都采用这种算法（通过改良缺点1）来回收新生代；
如Serial收集器、ParNew收集器、Parallel Scavenge收集器、、G1（从局部看）；

### 3.标记-整理算法
"标记-整理"（Mark-Compact）算法是根据老年代的特点提出的。

#### 算法思路
1. 标记：标记过程与"标记-清除"算法一样；
2. 整理：但后续不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动；然后直接清理掉端边界以外的内存；

#### 优点
1）不会像复制算法，效率随对象存活率升高而变低
老年代特点：
* 对象存活率高，没有额外的空间可以分配担保；
* 所以老年代一般不能直接选用复制算法算法，而选用标记-整理算法；

2）不会像标记-清除算法，产生内存碎片
因为清除前，进行了整理，存活对象都集中到空间一侧；

#### 缺点
主要是效率问题：除像标记-清除算法的标记过程外，还多了需要整理的过程，效率更低；

#### 应用场景
很多垃圾收集器采用这种算法来回收**老年代**，如Serial Old收集器、G1（从整体看）；

## 四、JVM垃圾收集器
先来了解HotSpot虚拟机中的7种垃圾收集器：Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS、G1，先介绍一些垃圾收集的相关概念，再介绍它们的主要特点、应用场景、以及一些设置参数和基本运行原理。

### 1. 垃圾收集器概述
垃圾收集器是垃圾回收算法（标记-清除算法、复制算法、标记-整理算法、火车算法）的具体实现，不同商家、不同版本的JVM所提供的垃圾收集器可能会有很在差别，本文主要介绍HotSpot虚拟机中的垃圾收集器。

### 2. 垃圾收集器组合
JDK7/8后，HotSpot虚拟机所有收集器及组合（连线），如下图：

![](https://github.com/TFdream/blog/blob/master/docs/image/JVM/Jvm_collector.png)

图中展示了7种不同分代的收集器：
* Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS、G1；

它们所处区域，则表明其是属于新生代收集器还是老年代收集器：

* 新生代收集器：Serial、ParNew、Parallel Scavenge；
* 老年代收集器：Serial Old、Parallel Old、CMS；
* 整堆收集器：G1；

两个收集器间有连线，表明它们可以搭配使用：

Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1；

其中Serial Old作为CMS出现"Concurrent Mode Failure"失败的后备预案（后面介绍）；

## 内存管理参数
JDK 1.6的Hotspot虚拟机有很多非稳定参数（），使用-XX:+PrintFlagFinal参数可以输出所有参数的名称及默认值。
参数使用的方式有如下3种：
* -XX:+\<option\> ：开启option参数
* -XX:\<option\> ：关闭option参数
* -XX:\<option\>=\<value\> ：将option参数的值设置为value

| 参数 | 默认值 | 使用介绍 |
| --- | --- | --- |
| DisableExplicitGC | 默认关闭 | 忽略来着System.gc()方法触发的垃圾收集|
| ExplicitGCInvokesConcurrent | 默认关闭 | 当收到System.gc()方法提交的垃圾收集请求时，使用CMS收集器进行收集|
| UseSerialGC | Client模式的虚拟机默认开启，其他模式关闭 | 虚拟机运行在Client模式的默认值，打开此开关后，使用Serial+Serial Old 的收集器组合进行内存回收|
| UseParNewGC | 默认关闭 | 打开此开关后，使用ParNew + Serial Old 的收集器组合进行内存回收|
| UseConcMarkSweepGC | 默认关闭 | 打开此开关后，使用ParNew + CMS + Serial Old 的收集器组合进行内存回收。如果CMS收集器出现Concurrent Mode Failure，则Serial Old收集器将作为后备收集器|
| UseParallelGC | Server模式的虚拟机默认开启，其他模式关闭 | 虚拟机运行在Server模式的默认值，打开此开关后，，使用Parallel Scavenge + Serial Old 的收集器组合进行内存回收|
| UseParallelOldGC | 默认关闭 | 打开此开关后，，使用Parallel Scavenge + Parallel Old 的收集器组合进行内存回收|

## OutOfMemoryError类型
上述区域中，除了程序计数器，其他在VM Spec中都描述了产生OutOfMemoryError（下称OOM）的情形，那我们就实战模拟一下，通过几段简单的代码，令对应的区域产生OOM异常以便加深认识，同时初步介绍一些与内存相关的虚拟机参数。下文的代码都是基于Sun Hotspot虚拟机1.6版的实现，对于不同公司的不同版本的虚拟机，参数与程序运行结果可能结果会有所差别。

### Java堆
Java堆存放的是对象实例，因此只要不断建立对象，并且保证GC Roots到对象之间有可达路径即可产生OOM异常。测试中限制Java堆大小为20M，不可扩展，通过参数```-XX:+HeapDumpOnOutOfMemoryError```让虚拟机在出现OOM异常的时候Dump出内存映像以便分析。

```
/**
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * @author zzm
 */
public class HeapOOM {

       static class OOMObject {

       }
       public static void main(String[] args) {
              List<OOMObject> list = new ArrayList<OOMObject>();
              while (true) {
                     list.add(new OOMObject());
              }
       }
}
```
运行结果：
```
java.lang.OutOfMemoryError: Java heap space

Dumping heap to java_pid3404.hprof ...

Heap dump file created [22045981 bytes in 0.663 secs]
```

### VM栈和本地方法栈

Hotspot虚拟机并不区分VM栈和本地方法栈，因此-Xoss参数实际上是无效的，栈容量只由-Xss参数设定。关于VM栈和本地方法栈在VM Spec描述了两种异常：StackOverflowError与OutOfMemoryError，当栈空间无法继续分配分配时，到底是内存太小还是栈太大其实某种意义上是对同一件事情的两种描述而已，在笔者的实验中，对于单线程应用尝试下面3种方法均无法让虚拟机产生OOM，全部尝试结果都是获得SOF异常。

1.使用-Xss参数削减栈内存容量。结果：抛出SOF异常时的堆栈深度相应缩小。
2.定义大量的本地变量，增大此方法对应帧的长度。结果：抛出SOF异常时的堆栈深度相应缩小。
3.创建几个定义很多本地变量的复杂对象，打开逃逸分析和标量替换选项，使得JIT编译器允许对象拆分后在栈中分配。结果：实际效果同第二点。

清单2：VM栈和本地方法栈OOM测试（仅作为第1点测试程序）：
```
/**
 * VM Args：-Xss128k
 * @author zzm
 */
public class JavaVMStackSOF {

       private int stackLength = 1;

       public void stackLeak() {
              stackLength++;
              stackLeak();

       }

       public static void main(String[] args) throws Throwable {
              JavaVMStackSOF oom = new JavaVMStackSOF();
              try {
                     oom.stackLeak();
              } catch (Throwable e) {
                     System.out.println("stack length:" + oom.stackLength);
                     throw e;

              }
       }
}
```
运行结果：
```
stack length:2402

Exception in thread "main" java.lang.StackOverflowError

        at org.fenixsoft.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:20)

        at org.fenixsoft.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:21)

        at org.fenixsoft.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:21)
```

如果在多线程环境下，不断建立线程倒是可以产生OOM异常，但是基本上这个异常和VM栈空间够不够关系没有直接关系，甚至是给每个线程的VM栈分配的内存越多反而越容易产生这个OOM异常。

原因其实很好理解，操作系统分配给每个进程的内存是有限制的，譬如32位Windows限制为2G，Java堆和方法区的大小JVM有参数可以限制最大值，那剩余的内存为2G（操作系统限制）-Xmx（最大堆）-MaxPermSize（最大方法区），程序计数器消耗内存很小，可以忽略掉，那虚拟机进程本身耗费的内存不计算的话，剩下的内存就供每一个线程的VM栈和本地方法栈瓜分了，那自然每个线程中VM栈分配内存越多，就越容易把剩下的内存耗尽。

清单3：创建线程导致OOM异常：
```
/**
 * VM Args：-Xss2M （这时候不妨设大些）
 * @author zzm
 */
public class JavaVMStackOOM {

       private void dontStop() {
              while (true) {

              }
       }

       public void stackLeakByThread() {
              while (true) {
                     Thread thread = new Thread(new Runnable() {
                            @Override

                            public void run() {
                                   dontStop();
                            }
                     });
                     thread.start();
              }

       }

       public static void main(String[] args) throws Throwable {
              JavaVMStackOOM oom = new JavaVMStackOOM();
              oom.stackLeakByThread();
       }
}
```

特别提示一下，如果读者要运行上面这段代码，记得要存盘当前工作，上述代码执行时有很大令操作系统卡死的风险。

运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

### 运行时常量池

要在常量池里添加内容，最简单的就是使用String.intern()这个Native方法。由于常量池分配在方法区内，我们只需要通过-XX:PermSize和-XX:MaxPermSize限制方法区大小即可限制常量池容量。实现代码如下：

清单4：运行时常量池导致的OOM异常：
```
/**
 * VM Args：-XX:PermSize=10M -XX:MaxPermSize=10M
 * @author zzm
 */
public class RuntimeConstantPoolOOM {

       public static void main(String[] args) {
              // 使用List保持着常量池引用，压制Full GC回收常量池行为
              List<String> list = new ArrayList<String>();

              // 10M的PermSize在integer范围内足够产生OOM了

              int i = 0;
              while (true) {
                     list.add(String.valueOf(i++).intern());
              }
       }

}
```

运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space

       at java.lang.String.intern(Native Method)

       at org.fenixsoft.oom.RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java:18)
```

### 方法区

上文讲过，方法区用于存放Class相关信息，所以这个区域的测试我们借助CGLib直接操作字节码动态生成大量的Class，值得注意的是，这里我们这个例子中模拟的场景其实经常会在实际应用中出现：当前很多主流框架，如Spring、Hibernate对类进行增强时，都会使用到CGLib这类字节码技术，当增强的类越多，就需要越大的方法区用于保证动态生成的Class可以加载入内存。

清单5：借助CGLib使得方法区出现OOM异常：
```
/**
 * VM Args： -XX:PermSize=10M -XX:MaxPermSize=10M
 * @author zzm
 */
public class JavaMethodAreaOOM {

       public static void main(String[] args) {
              while (true) {
                     Enhancer enhancer = new Enhancer();
                     enhancer.setSuperclass(OOMObject.class);
                     enhancer.setUseCache(false);
                     enhancer.setCallback(new MethodInterceptor() {

                            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {

                                   return proxy.invokeSuper(obj, args);

                            }

                     });
                     enhancer.create();
              }
       }

       static class OOMObject {

       }
}
```

运行结果：
```
Caused by: java.lang.OutOfMemoryError: PermGen space

       at java.lang.ClassLoader.defineClass1(Native Method)

       at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)

       at java.lang.ClassLoader.defineClass(ClassLoader.java:616)

       ... 8 more
```

### 本机直接内存

DirectMemory容量可通过-XX:MaxDirectMemorySize指定，不指定的话默认与Java堆（-Xmx指定）一样，下文代码越过了DirectByteBuffer，直接通过反射获取Unsafe实例进行内存分配（Unsafe类的getUnsafe()方法限制了只有引导类加载器才会返回实例，也就是基本上只有rt.jar里面的类的才能使用），因为DirectByteBuffer也会抛OOM异常，但抛出异常时实际上并没有真正向操作系统申请分配内存，而是通过计算得知无法分配既会抛出，真正申请分配的方法是unsafe.allocateMemory()。

```
/**
 * VM Args：-Xmx20M -XX:MaxDirectMemorySize=10M
 * @author zzm
 */
public class DirectMemoryOOM {

       private static final int _1MB = 1024 * 1024;

       public static void main(String[] args) throws Exception {
              Field unsafeField = Unsafe.class.getDeclaredFields()[0];
              unsafeField.setAccessible(true);
              Unsafe unsafe = (Unsafe) unsafeField.get(null);
              while (true) {
                     unsafe.allocateMemory(_1MB);
              }
       }
}
```

运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError

       at sun.misc.Unsafe.allocateMemory(Native Method)

       at org.fenixsoft.oom.DirectMemoryOOM.main(DirectMemoryOOM.java:20)
```


## 性能优化
* [Java性能优化指南1.8版，及唯品会的实战](http://calvin1978.blogcn.com/articles/javatuning.html)
* [一份平民化的应用性能优化检查列表（完整篇）](http://calvin1978.blogcn.com/articles/checklist.html)

## 参考资料
* Java虚拟机垃圾回收(二) 垃圾回收算法：https://blog.csdn.net/tjiyu/article/details/53983064
* Java虚拟机垃圾回收(三) 7种垃圾收集器：https://blog.csdn.net/tjiyu/article/details/53983650

