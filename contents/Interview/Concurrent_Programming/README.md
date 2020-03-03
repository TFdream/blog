# 并发编程

## 线程
### 线程的生命周期
Java语言中定义了5种线程状态，在任意一个时间点，一个线程只能有且只有其中一种状态，这5种状态是：

- 新建（New）：创建后尚未启动的线程处于这种状态。
- 运行（Runable）：包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。
- 无限期等待（Waiting）：处于这种状态的线程不会被分配CPU执行时间，它们要等待被其他线程显式地唤醒。以下方法会让线程陷入无限期的等待状态：
  - 没有设置timeout参数的Object.wait()方法；
  - 没有设置timeout参数的Thread.join()方法；
  - LockSupport.park()方法；
- 限期等待（Timed Waiting）：处于这种状态的线程也不会被分配CPU执行时间，不过无须等待被其他线程显式地唤醒，在一定时间之后它们会由操作系统自动唤醒。以下方法会让线程进入限期等待状态：
  - Thread.sleep()方法；
  - 设置了timeout参数的Object.wait()方法；
  - 设置了timeout参数的Thread.join()方法；
  - LockSupport.parkNanos()方法；
  - LockSupport.parkUntil()方法；
- 阻塞（Blocked）：线程被阻塞了，“阻塞状态”与“等待状态”的区别是：“阻塞状态”在等待着获取到一个排它锁，这个事件将在另外一个线程放弃这个锁的时候发生；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域（synchronized）的时候，线程将进入这种状态。
- 结束（Terminated）：已终止的线程状态，线程已经结束执行。

#### 线程间的状态转换
#### 1、新建(New)
新创建了一个线程对象，还未调用start()方法。
```
Thread thread = new Thread();
```
#### 2、就绪（Ready）
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中 获取cpu 的使用权 。

#### 3、运行中（Running）
可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

#### 4、限期等待（Timed Waiting）
也可以称作 TIMED_WAITING（有等待时间的等待状态）。

线程主动调用以下方法：
* Thread.sleep方法；
* Object的wait方法，带有时间；
* Thread.join方法，带有时间；
* LockSupport的parkNanos方法，带有时间。

#### 5、无限期等待（Waiting）
运行中（Running）的线程执行了以下方法：
* Object的wait方法，并且没有使用timeout参数; 
* Thread的join方法，没有使用timeout参数；
* LockSupport的park方法；
* Conditon的await方法。

#### 6、阻塞（Blocked）
阻塞状态是指线程因为某种原因放弃了cpu 使用权，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分两种：

* 同步阻塞：运行(running)的线程进入了一个synchronized方法，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。
* 其他阻塞：运行(running)的线程发出了I/O请求时，JVM会把该线程置为阻塞状态。当I/O处理完毕时，线程重新转入可运行(runnable)状态。

#### 7、结束（Terminated）
线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。

具体参考：[Java线程的状态](https://www.jianshu.com/p/dbbcceb6bc2a)

### 创建线程的方式
两种方式：
* 通过实现java.lang.Runnable
* 通过扩展java.lang.Thread类.

相比扩展Thread，实现Runnable接口可能更优，原因有二：
* Java不支持多继承，因此扩展Thread类就代表这个子类不能扩展其他类，而实现Runnable接口的类还可能扩展另一个类；
* 类可能只要求可执行即可，因此继承自Thread类的开销过大。

### Runnable和Callable的区别
Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已； Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

### Thread的 sleep() 、join()、yield() 有什么区别

Thread.sleep() 使当前线程（即调用sleep方法的线程暂停一段时间），给其它的线程运行的机会，不考虑其它线程的优先级的，并且不释放资源锁，也就是说如果有synchronized同步块，其它线程仍然是不能访问共享数据的；

Thread.yield() 使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程转入到就绪状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程。

Thread.join 把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的join()方法，那么直到线程A执行完毕后，才会继续执行线程B。

### 线程池的实现原理

具体参考：[ThreadPoolExecutor 线程池源码分析](https://juejin.im/post/5a7abe75f265da4e7e10a19e)

### 如何合理配置核心线程数？
分析下线程池处理的程序是CPU密集型，还是IO密集型。

《java并发编程实战》，里面有讲，要正确地设置线程池的大小，你必须估算出任务的等待时间和计算时间的比值，这种估算不会很精确。也可以使用另一种方式，就是：在某一个基准负载下，分别设置不同大小的线程池来运行应用程序，并观察cpu利用率的水平。

**线程数=cpu可用核心数/（1-阻塞系数）**，其中阻塞系数的取值在[0,1]之间。计算密集型任务的阻塞系数为0，而IO密集型任务的阻塞系数则接近1。一般，我们让线程执行的任务是比较复杂的，不会是单一的计算密集型任务，或者单一的IO密集型任务，通常会夹杂着。那么就需要我们去计算阻塞系数了。**阻塞系数的定义就是执行该任务阻塞的时间与（阻塞时间+计算时间）的比值，也就是w/(w+c)**。

CPU密集型：核心线程数 = CPU核数 + 1

IO密集型：核心线程数 = CPU核数 * 2

### ThreadLocal实现原理
简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal。ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了。

具体参考：[ThreadLocal源码分析](http://blog.csdn.net/top_code/article/details/51397397)

## 锁机制
### Java中的有哪些锁
比如Synchronized和ReentrantLock。

### synchronized的实现原理
Synchronized是JVM实现的一种锁，其中锁的获取和释放分别是monitorenter和monitorexit指令，该锁在实现上分为了偏向锁、轻量级锁和重量级锁，其中偏向锁在1.6是默认开启的，轻量级锁在多线程竞争的情况下会膨胀成重量级锁，有关锁的数据都保存在对象头中。

### ReentrantLock的实现原理
ReentrantLock是基于AQS实现的。

### 什么是AQS
在AQS内部会保存一个状态变量state，通过CAS修改该变量的值，修改成功的线程表示获取到该锁，没有修改成功，或者发现状态state已经是加锁状态，则通过一个Waiter对象封装线程，添加到等待队列中，并挂起等待被唤醒。

### CAS的实现原理
CAS（Compare and Swap）是通过Unsafe类的compareAndSwap方法实现的，第一个参数是要修改的对象，第二个参数是对象中要修改变量的偏移量，第三个参数是修改之前的值，第四个参数是预想修改后的值。

### CAS缺点
CAS存在一个很明显的问题，即ABA问题。
问题：如果变量V初次读取的时候是A，并且在准备赋值的时候检查到它仍然是A，那能说明它的值没有被其他线程修改过了吗？
如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性。

### AtomicInteger 内部实现
其实就是 CAS + volatile

### synchronized 与 lock 的区别
ReentrantLock 在加锁和内存上提供的语义上与内置锁相同，此外它还提供了一些其他功能，包括定时的锁等待、可中断的锁等待、公平性，以及实现非块结构的加锁。

### 乐观锁&悲观锁


## CountDownLatch
在多线程协作完成业务功能时，有时候需要等待其他多个线程完成任务之后，主线程才能继续往下执行业务功能，在这种的业务场景下，通常可以使用Thread类的join方法，让主线程等待被join的线程执行完之后，主线程才能继续往下执行。当然，使用线程间消息通信机制也可以完成。其实，java并发工具类中为我们提供了类似“倒计时”这样的工具类，可以十分方便的完成所说的这种业务场景。

下面来看些CountDownLatch的一些重要方法：
```
//调用该方法的线程等到构造方法传入的N减到0的时候，才能继续往下执行；
await() throws InterruptedException;

//与上面的await方法功能一致，只不过这里有了时间限制，调用该方法的线程等到指定的timeout时间后，不管N是否减至为0，都会继续往下执行；
await(long timeout, TimeUnit unit);

//使CountDownLatch初始值N减1；
countDown();

//获取当前CountDownLatch维护的值；
long getCount();

```

CountDownLatch的构造方法看起：
```
public CountDownLatch(int count)
```
构造方法会传入一个整型数N，之后调用CountDownLatch的countDown方法会对N减一，知道N减到0的时候，当前调用await方法的线程继续执行。


**另外，需要注意的是，当调用CountDownLatch的countDown方法时，当前线程是不会被阻塞，会继续往下执行**。

## CyclicBarrier
CyclicBarrier也是一种多线程并发控制的实用工具，和CountDownLatch一样具有等待计数的功能，但是相比于CountDownLatch功能更加强大。

为了理解CyclicBarrier，这里举一个通俗的例子。开运动会时，会有跑步这一项运动，我们来模拟下运动员入场时的情况，假设有6条跑道，在比赛开始时，就需要6个运动员在比赛开始的时候都站在起点了，裁判员吹哨后才能开始跑步。跑道起点就相当于“barrier”，是临界点，而这6个运动员就类比成线程的话，就是这6个线程都必须到达指定点了，意味着凑齐了一波，然后才能继续执行，否则每个线程都得阻塞等待，直至凑齐一波即可。cyclic是循环的意思，也就是说CyclicBarrier当多个线程凑齐了一波之后，仍然有效，可以继续凑齐下一波。

下面来看下CyclicBarrier的主要方法：
```
//等到所有的线程都到达指定的临界点
await() throws InterruptedException, BrokenBarrierException 

//与上面的await方法功能基本一致，只不过这里有超时限制，阻塞等待直至到达超时时间为止
await(long timeout, TimeUnit unit) throws InterruptedException, 
BrokenBarrierException, TimeoutException 

//获取当前有多少个线程阻塞等待在临界点上
int getNumberWaiting()

//用于查询阻塞等待的线程是否被中断
boolean isBroken()

	
//将屏障重置为初始状态。如果当前有线程正在临界点等待的话，将抛出BrokenBarrierException。
void reset()

```

另外需要注意的是，CyclicBarrier提供了这样的构造方法：
```
public CyclicBarrier(int parties, Runnable barrierAction)
```
可以用来，当指定的线程都到达了指定的临界点的时，接下来执行的操作可以由barrierAction传入即可。

## CountDownLatch与CyclicBarrier的比较
CountDownLatch与CyclicBarrier都是用于控制并发的工具类，都可以理解成维护的就是一个计数器，但是这两者还是各有不同侧重点的：
* CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；CountDownLatch强调一个线程等多个线程完成某件事情。CyclicBarrier是多个线程互等，等大家都完成，再携手共进。
* 调用CountDownLatch的countDown方法后，当前线程并不会阻塞，会继续往下执行；而调用CyclicBarrier的await方法，会阻塞当前线程，直到CyclicBarrier指定的线程全部都到达了指定点的时候，才能继续往下执行；
* CountDownLatch方法比较少，操作比较简单，而CyclicBarrier提供的方法更多，比如能够通过getNumberWaiting()，isBroken()这些方法获取当前多个线程的状态，并且CyclicBarrier的构造方法可以传入barrierAction，指定当所有线程都到达时执行的业务功能；
* CountDownLatch是不能复用的，而CyclicLatch是可以复用的。

## Semaphore
Semaphore可以理解为信号量，用于控制资源能够被并发访问的线程数量，以保证多个线程能够合理的使用特定资源。Semaphore就相当于一个许可证，线程需要先通过acquire方法获取该许可证，该线程才能继续往下执行，否则只能在该方法出阻塞等待。当执行完业务功能后，需要通过release()方法将许可证归还，以便其他线程能够获得许可证继续执行。
Semaphore可以用于做流量控制，特别是公共资源有限的应用场景，比如数据库连接。假如有多个线程读取数据后，需要将数据保存在数据库中，而可用的最大数据库连接只有10个，这时候就需要使用Semaphore来控制能够并发访问到数据库连接资源的线程个数最多只有10个。在限制资源使用的应用场景下，Semaphore是特别合适的。

下面来看下Semaphore的主要方法：
```
//获取许可，如果无法获取到，则阻塞等待直至能够获取为止
void acquire() throws InterruptedException 

//同acquire方法功能基本一样，只不过该方法可以一次获取多个许可
void acquire(int permits) throws InterruptedException

//释放许可
void release()

//释放指定个数的许可
void release(int permits)

//尝试获取许可，如果能够获取成功则立即返回true，否则，则返回false
boolean tryAcquire()

//与tryAcquire方法一致，只不过这里可以指定获取多个许可
boolean tryAcquire(int permits)

//尝试获取许可，如果能够立即获取到或者在指定时间内能够获取到，则返回true，否则返回false
boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException

//与上一个方法一致，只不过这里能够获取多个许可
boolean tryAcquire(int permits, long timeout, TimeUnit unit)

//返回当前可用的许可证个数
int availablePermits()

```

## ConcurrentHashMap实现原理
[谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)

## 生产者 — 消费者问题
生产者-消费者模式是一个十分经典的多线程并发协作的模式，弄懂生产者-消费者问题能够让我们对并发编程的理解加深。所谓生产者-消费者问题，实际上主要是包含了两类线程，一种是生产者线程用于生产数据，另一种是消费者线程用于消费数据，为了解耦生产者和消费者的关系，通常会采用共享的数据区域，就像是一个仓库，生产者生产数据之后直接放置在共享数据区中，并不需要关心消费者的行为；而消费者只需要从共享数据区中去获取数据，就不再需要关心生产者的行为。但是，这个共享数据区域中应该具备这样的线程间并发协作的功能：
* 如果共享数据区已满的话，阻塞生产者继续生产数据放置入内；
* 如果共享数据区为空的话，阻塞消费者继续消费数据；

在实现生产者消费者问题时，可以采用三种方式：
* 使用Object的wait/notify的消息通知机制；
* 使用Lock的Condition的await/signal的消息通知机制；
* 使用BlockingQueue实现。

本文主要将这三种实现方式进行总结归纳。

[Java实现生产者-消费者模型的几种方法](https://www.jianshu.com/p/9f6b7fb891dd)


## volatile关键字
volatile关键字提供了内存可见性和禁止内存重排序。
因为在虚拟机内存中有主内存和工作内存的概念，每个cpu都有自己的工作内存，当读取一个普通变量时，优先读取工作内存的变量，如果工作内存中没有对应的变量，则从主内存中加载到工作内存，对工作内存的普通变量进行修改，不会立马同步到主内存，内存可见性保证了在多线程的场景下，保证了线程A对变量的修改，其它线程可以读到最新值&&%%……

当对volatile修饰的变量进行写操作时，直接把最新值写到主内存中，并清空其它cpu工作内存中该变量所在的内存行数据，当对volatile修饰的变量进行读操作时，会读取主内存的数据

## 有哪些多线程开发良好的实践?
* 给线程命名；
* 最小化同步范围；
* 优先使用volatile；
* 尽可能使用更高层次的并发工具而非wait和notify()来实现线程通信，如BlockingQueue,Semeaphore；
* 优先使用并发容器而非同步容器；
* 考虑使用线程池

## happen-before原则
先行发生（happen-before）原则：

1. **程序次序规则**：在一个线程内，书写在前面的操作happen-before书写在后面的操作。这条规则是说，在单线程 中操作间happen-before关系完全是由源代码的顺序决定的，这里的前提“在同一个线程中”是很重要的，这条规则也称为单线程规则 。这个规则多少说得有些简单了，考虑到控制结构和循环结构，书写在后面的操作可能happen-before书写在前面的操作，不过我想读者应该明白我的意思。 
2. **管程锁定规则**：对锁的unlock操作happen-before后续的对同一个锁的lock操作。这里的“后续”指的是时间上的先后关系，unlock操作发生在退出同步块之后，lock操作发生在进入同步块之前。这是条最关键性的规则，线程安全性主要依赖于这条规则。但是仅仅是这条规则仍然不起任何作用，它必须和下面这条规则联合起来使用才显得意义重大。这里关键条件是必须对“同一个锁”的lock和unlock。 如果操作A happen-before操作B，操作B happen-before操作C，那么操作A happen-before操作C。这条规则也称为传递规。
3. **volatile变量规则**：对volatile字段的写操作happen-before后续的对同一个字段的读操作.（Java5 新增）。
4. **线程启动规则**：Thread对象的start()方法先行发生于此线程的每一个动作。
5. **线程终止规则**：线程中的所有操作都先行发生于此线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()方法的返回值等手段检测到线程已经终止执行。
6. **线程中断规则**：对线程的interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件发生，可以通过Thread.interrupted()方法检测到是否有中断发生。
7. **对象终结规则**：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
8. **传递性**：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

Java语言无须任何同步手段保障就能成立的先行发生规则就只有上面这些了。



本文将会不定期更新，欢迎大家持续关注！


