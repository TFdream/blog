## 线程的生命周期
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

## 线程间的状态转换
### 1、新建(New)
新创建了一个线程对象，还未调用start()方法。
```
Thread thread = new Thread();
```
### 2、就绪（Ready）
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中 获取cpu 的使用权 。

### 3、运行中（Running）
可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

### 4、限期等待（Timed Waiting）
也可以称作 TIMED_WAITING（有等待时间的等待状态）。

线程主动调用以下方法：
* Thread.sleep方法；
* Object的wait方法，带有时间；
* Thread.join方法，带有时间；
* LockSupport的parkNanos方法，带有时间。

### 5、无限期等待（Waiting）
运行中（Running）的线程执行了以下方法：
* Object的wait方法，并且没有使用timeout参数; 
* Thread的join方法，没有使用timeout参数；
* LockSupport的park方法；
* Conditon的await方法。

### 6、阻塞（Blocked）
阻塞状态是指线程因为某种原因放弃了cpu 使用权，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得cpu timeslice 转到运行(running)状态。阻塞的情况分两种：

* 同步阻塞：运行(running)的线程进入了一个synchronized方法，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。
* 其他阻塞：运行(running)的线程发出了I/O请求时，JVM会把该线程置为阻塞状态。当I/O处理完毕时，线程重新转入可运行(runnable)状态。

### 7、结束（Terminated）
线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。


-----
结束

