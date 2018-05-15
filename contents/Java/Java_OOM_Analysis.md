## 背景
前段时间系统经常出现OOM，每次出现之后系统会出现各种问题，临时解决方案只能是重启，然后等找到问题后再发布解决。

## 问题定位
线上问题日志如下：
```
Exception in thread "msgWorkTP-811568603-1-thread-6" java.lang.OutOfMemoryError: Java heap space

Exception in thread "schedulerFactory_QuartzSchedulerThread" java.lang.OutOfMemoryError: Java heap space

Exception in thread "server-timer" java.lang.OutOfMemoryError: Java heap space

Exception in thread "Tracer-AsyncAppender-Thread-CommonAppender" java.lang.OutOfMemoryError: Java heap space
```

线上遇到OOM需要做两件事情第一个是dump内存，第二个是看下GC日志。 

### 1. dump内存 
使用jmap命令dump内存，需要注意的是，在linux JDK1.6某个版本里使用jmap可能会让系统挂掉，可以通过-d64来解决。
```
jmap -J-d64 -dump:format=b,file=dump.bin PID
```
一般dump下来的内存有几个G，而我这次在线上dump下来只有200M，说明jmap有问题，这个时候可以用gcore 把整个内存dump出来，然后再使用jmap把core dump转换成heap dump。命令如下：

```
$ jmap  -permstat  /opt/taobao/java/bin/java core.17024
```
因为我们没有线上权限，又担心dump的时候系统容易挂掉，所以我们在JVM里加了个参数，在OOM的时候自动dump内存，
```
-XX:+HeapDumpOnOutOfMemeryError
```

### 2. 查看gc日志

如果没有任何JVM参数设置，gc日志默认打印在stdout.log文件里，里面可能会打其他的日志，而且GC日志也不会输出时间，所以在JVM启动参数里最好加以下命令，规范下GC日志输出到/home/admin/logs/gc.log，并且打印GC时间。
```
-XX:HeapDumpPath=/home/admin/logs -Xloggc:/home/admin/logs/gc.log  -XX:+PrintGCDetails -XX:+PrintGCDateStamps
```

在查看GC日志里发现concurrent mode failure问题，日志如下：

(concurrent mode failure): 1146697K->1146697K(1146880K), 1.0353630 secs] 1773385K->1697954K(1773568K),

这个问题是因为CMS（Concurrent Mark-Sweep）垃圾回收器在做fullgc，还没到达回收的阶段，但是年轻代又有新对象晋升到年老代，而年老代又没有足够空间了，只能全停机，进行fullgc。但是从日志上看没有释放多少空间，要么是堆不够大，要么是泄露，先调大堆，如果是泄露，还是会重现，如果不是泄露，就稳定了。于是我们进行了JVM参数调整,系统有8G内存，我们把JVM堆内存升到3.8G，修改参数XX:CMSInitiatingOccupancyFraction=60，让年老代在达到60%的时候进行CMS回收，这样在进行回收时候年老贷至少还有3800x0.4=1520m空间，就算把整个年轻代塞进去也没有问题。
```
-server -Xms3800m -Xmx3800m -Xmn1500m -Xss256k -XX:PermSize=340m -XX:MaxPermSize=340m -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60 -XX:+CMSClassUnloadingEnabled -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs -Xloggc:/home/admin/logs/gc.log -Dcom.sun.management.jmxremote.port=9981 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dfile.encoding=UTF-8
```

参数调整后，过了几天还是出现OOM，可以断定是内存溢出导致的，通过自动dump下来的内存文件很快发现有一个对象占用内存非常大，解决后系统恢复正常。

### 3. 使用VisualVM分析


### 4. 使用MAT分析


## 参考资料
* [又一次线上OOM排查经过 - ImportNew](http://www.importnew.com/24393.html)
* [一次应用OOM排查 - 并发编程网](http://ifeve.com/one-java-oom/)
