## 问题背景
昨天下午突然收到运维邮件报警，显示数据平台服务器cpu利用率达到了98.94%，而且最近一段时间一直持续在70%以上，看起来像是硬件资源到瓶颈需要扩容了，但仔细思考就会发现咱们的业务系统并不是一个高并发或者CPU密集型的应用，这个利用率有点太夸张，硬件瓶颈应该不会这么快就到了，一定是哪里的业务代码逻辑有问题。

## 排查思路
传统的方案一般是4步：
* top oder by with P // 首先按进程负载排序找到  maxLoad(pid)
* top -Hp // 找到相关负载 线程PID
* printf “0x%x\n”线程PID // 将线程PID转换为 16进制，为后面查找 jstack 日志做准备
* jstack 进程PID | grep 十六进制线程PID  // 例如：jstack 1040 | grep 0x431

但是对于线上问题定位来说，分秒必争，上面的 4 步还是太繁琐耗时了，之前介绍过淘宝的[oldratlee]() 同学就将上面的流程封装为了一个工具：[show-busy-java-threads.sh](https://github.com/oldratlee/useful-scripts)，可以很方便的定位线上的这类问题。

## 参考资料
* [useful-scripts - 淘宝李鼎](https://github.com/oldratlee/useful-scripts)
* [awesome-scripts](https://github.com/Suishenyun/awesome-scripts)
* [jvm-tools](https://github.com/aragozin/jvm-tools)
