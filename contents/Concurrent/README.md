# 多线程&并发编程

## 1、说说进程、线程、协程之间的区别
简而言之，进程是程序运行和资源分配的基本单位，一个程序至少有一个进程,一个进程至少有一个线程。进程在执行过程中拥有独立的内存单元，而多个线程共享内存资源，减少切换次数，从而效率更高。

线程是进程的一个实体，是cpu调度和分派的基本单位,是比程序更小的能独立运行的基本单位。同一进程中的多个线程之间可以并发执行。

## 2、什么是守护线程?它和非守护线程有什么区别
程序运行完毕，JVM会等待非守护线程完成后关闭，但是jvm不会等待守护线程。守护线程最典型的例子就是GC线程。


## 3、线程的状态有哪些
请参考我的另外一篇文章：Java 线程的状态及切换

## 4、创建两种线程的方式?他们有什么区别?
* 通过实现java.lang.Runnable
* 通过扩展java.lang.Thread类.

相比扩展Thread,实现Runnable接口可能更优，原因有二:
* Java不支持多继承，因此扩展Thread类就代表这个子类不能扩展其他类.而实现Runnable接口的类还可能扩展另一个类。
* 类可能只要求可执行即可，因此集成整个Thread类的开销过大。

## 5、Runnable和Callable的区别
Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已； Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

这其实是很有用的一个特性，因为多线程相比单线程更难、更复杂的一个重要原因就是因为多线程充满着未知性， 某条线程是否执行了？某条线程执行了多久？某条线程执行的时候我们期望的数据是否已经赋值完毕？无法得知，我们能做的只是等待这条多线程的任务执行完毕而已。 而Callable+Future/FutureTask却可以获取多线程运行的结果，可以在等待时间太长没获取到需要的数据的情况下取消该线程的任务，真的是非常有用。

## 6、Thread yield和join 区别
Thread.yield() 使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程。

Thread.join 把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的join()方法，那么直到线程A执行完毕后，才会继续执行线程B。

## 7、synchronized和ReentrantLock的区别
synchronized是和if、else、for、while一样的关键字，ReentrantLock是类，这是二者的本质区别。 既然ReentrantLock是类，那么它就提供了比synchronized更多更灵活的特性，可以被继承、可以有方法、可以有各种各样的类变量，ReentrantLock比synchronized的扩展性体现在几点上：
* ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁
* ReentrantLock可以获取各种锁的信息
* ReentrantLock可以灵活地实现多路通知

另外，二者的锁机制其实也是不一样的：ReentrantLock底层调用的是Unsafe的park方法加锁，synchronized操作的应该是对象头中mark word。

## 8、AtomicInteger 内部实现
其实就是 CAS + volatile，参考：Java AtomicInteger原理分析


## 9、如何在两个线程间共享数据
通过在线程之间共享对象就可以了，然后通过wait/notify/notifyAll、await/signal/signalAll进行唤起和等待，比方说阻塞队列BlockingQueue就是为线程之间共享数据而设计的。


## 10、ThreadLoal 实现原理
简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal。ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了。

详细参考：ThreadLocal源码深入分析

## 11、ThreadPoolExecutor 构造参数有哪些？各代表什么意义？

## 12、ConcurrentHashMap 实现原理

## 13、volatile关键字的作用
简单的说，就是当你写一个 volatile 变量之前，Java 内存模型会插入一个写屏障（write barrier），读一个 volatile 变量之前，会插入一个读屏障（read barrier）。 意思就是说，在你写一个 volatile 域时，能保证任何线程都能看到你写的值，同时，在写之前，也能保证任何数值的更新对所有线程是可见的，因为内存屏障会将其他所有写的值更新到缓存。
volatile关键字可以保证 可见性和 禁止指令重排序。

在Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model，JMM）来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。Java语言本身对 **原子性**、**可见性**以及**有序性**。

## 14、CyclicBarrier和CountDownLatch区别
这两个类非常类似，都在java.util.concurrent下，都可以用来表示代码运行到某个点上，二者的区别在于：
* CyclicBarrier的某个线程运行到某个点上之后，该线程即停止运行，直到所有的线程都到达了这个点，所有线程才重新运行；CountDownLatch则不是，某线程运行到某个点上之后，只是给某个数值-1而已，该线程继续运行
* CyclicBarrier只能唤起一个任务，CountDownLatch可以唤起多个任务
* CyclicBarrier可重用，CountDownLatch不可重用，计数值为0该CountDownLatch就不可再用了

## 15、有哪些多线程开发良好的实践?
* 给线程命名；
* 最小化同步范围；
* 优先使用volatile；
* 尽可能使用更高层次的并发工具而非wait和notify()来实现线程通信，如BlockingQueue,Semeaphore；
* 优先使用并发容器而非同步容器；
* 考虑使用线程池


