
## 秒杀系统的设计
* [淘宝大秒系统设计详解-许令波](https://mp.weixin.qq.com/s?__biz=MzI3MzEzMDI1OQ==&mid=2651815451&idx=1&sn=141001dbadf3efc5f791735496d4a329&chksm=f0dc2867c7aba1714878dbc2ab1a33eac7fc5933f072b31e37a93071af485d06cb24774a2e25&mpshare=1&scene=1&srcid=11220PBqTM2PrSZL09WmzT1r)
* [秒杀系统架构优化思路](https://mp.weixin.qq.com/s/5aMN9SqaWa57rYGgtdAF_A)

## JVM性能调优

## JDK 1.8新特性

## CPU 使用率100%排查
某服务器上部署了若干tomcat实例，即若干垂直切分的Java站点服务，以及若干Java微服务，突然收到运维的CPU异常告警。

问：如何定位是哪个服务进程导致CPU过载，哪个线程导致CPU过载，哪段代码导致CPU过载？

### 步骤一、找到最耗CPU的进程
工具：top

方法：
* 执行top -c ，显示进程运行信息列表
* 键入P (大写p)，进程按照CPU使用率排序

图示：

如上图，最耗CPU的进程PID为10765

### 步骤二：找到最耗CPU的线程
工具：top

方法：
* top -Hp 10765 ，显示一个进程的线程运行信息列表
* 键入P (大写p)，线程按照CPU使用率排序

图示：

如上图，进程10765内，最耗CPU的线程PID为10804

### 步骤三：将线程PID转化为16进制

工具：printf

方法：printf “%x” 10804

图示：

如上图，10804对应的16进制是0x2a34，当然，这一步可以用计算器。

之所以要转化为16进制，是因为堆栈里，线程id是用16进制表示的。

### 步骤四：查看堆栈，找到线程在干嘛

工具：pstack/jstack/grep

方法：jstack 10765 | grep ‘0x2a34’ -C5 --color

* 打印进程堆栈
* 通过线程id，过滤得到线程堆栈

图示：

如上图，找到了耗CPU高的线程对应的线程名称“AsyncLogger-1”，以及看到了该线程正在执行代码的堆栈。


## OOM问题排查

## 领域驱动设计(DDD)

## 关注的技术领域

## 参考资料
* [linux cpu占用100%排查](https://blog.csdn.net/qinshi501/article/details/77442770)

