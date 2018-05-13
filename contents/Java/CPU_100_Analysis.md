## 背景
某服务器上部署了若干tomcat实例，即若干垂直切分的Java站点服务，以及若干Java微服务，突然收到运维的CPU异常告警。

问：如何定位是哪个服务进程导致CPU过载，哪个线程导致CPU过载，哪段代码导致CPU过载？

##  定位异常线程及具体代码行
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

## 总结
传统的方案一般是4步：
* top oder by with P // 首先按进程负载排序找到  maxLoad(pid)
* top -Hp // 找到相关负载 线程PID
* printf “0x%x\n”线程PID // 将线程PID转换为 16进制，为后面查找 jstack 日志做准备
* jstack 进程PID | grep 十六进制线程PID  // 例如：jstack 1040 | grep '0x431'

但是对于线上问题定位来说，分秒必争，上面的 4 步还是太繁琐耗时了，之前介绍过淘宝的[oldratlee]() 同学就将上面的流程封装为了一个工具：[show-busy-java-threads.sh](https://github.com/oldratlee/useful-scripts)，可以很方便的定位线上的这类问题。

## 参考资料
* [useful-scripts - 淘宝李鼎](https://github.com/oldratlee/useful-scripts)
* [awesome-scripts](https://github.com/Suishenyun/awesome-scripts)
* [jvm-tools](https://github.com/aragozin/jvm-tools)

