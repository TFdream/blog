# 并发篇

## 线程
### 线程的生命周期
Java线程一共有七个状态，分别是新建（New），就绪（Ready to run），运行中（Running），睡眠（Sleeping），阻塞（Blocked），等待（Waiting），死亡（Dead/Terminate）。

#### 线程间的状态转换
#### 1、新建(New)
新创建了一个线程对象，还未调用start()方法。
```
Thread thread = new Thread();
```
#### 2、就绪（Ready to run）
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中 获取cpu 的使用权 。

#### 3、运行中（Running）
可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

#### 4、睡眠（Sleeping）
也可以称作 TIMED_WAITING（有等待时间的等待状态）。

线程主动调用 这几个方法：1. Thread.sleep方法 2. Object的wait方法，带有时间 3. Thread.join方法，带有时间，4. LockSupport的parkNanos方法，带有时间。

#### 5、阻塞（Blocked）
阻塞状态是指线程因为某种原因放弃了cpu 使用权，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分两种：

同步阻塞：运行(running)的线程进入了一个synchronized方法，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。
其他阻塞：运行(running)的线程发出了I/O请求时，JVM会把该线程置为阻塞状态。当I/O处理完毕时，线程重新转入可运行(runnable)状态。
#### 6、等待（Waiting）
运行中（Running）的线程执行了4个方法中的任意方法： 1. Object的wait方法，并且没有使用timeout参数; 2. Thread的join方法，没有使用timeout参数 3. LockSupport的park方法，4:Conditon的await方法。

#### 7、死亡(dead)
线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。

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

### ThreadLocal实现原理
简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal。ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了。

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


## volatile关键字
volatile关键字提供了内存可见性和禁止内存重排序。
因为在虚拟机内存中有主内存和工作内存的概念，每个cpu都有自己的工作内存，当读取一个普通变量时，优先读取工作内存的变量，如果工作内存中没有对应的变量，则从主内存中加载到工作内存，对工作内存的普通变量进行修改，不会立马同步到主内存，内存可见性保证了在多线程的场景下，保证了线程A对变量的修改，其它线程可以读到最新值&&%%……

当对volatile修饰的变量进行写操作时，直接把最新值写到主内存中，并清空其它cpu工作内存中该变量所在的内存行数据，当对volatile修饰的变量进行读操作时，会读取主内存的数据


## ConcurrentHashMap实现原理
[谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)


