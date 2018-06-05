## 分布式算法
### Paxos

### Raft

### Zookeeper 原理
Zookeeper的角色
* 领导者（leader），负责进行投票的发起和决议，更新系统状态
* 学习者（learner），包括跟随者（follower）和观察者（observer），follower用于接受客户端请求并想客户端返回结果，在选主过程中参与投票
* Observer可以接受客户端连接，将写请求转发给leader，但observer不参加投票过程，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度
* 客户端（client），请求发起方

Zookeeper的核心是原子广播，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。

为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。

## 分布式事务一致性

### 2PC
* [分布式事务2PC && 3PC](http://int64.me/2016/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A12PC%20&&%203PC.html)

### TCC
* [分布式事务解决方案与适用场景分析](https://juejin.im/post/5a9cb1bf6fb9a028c06a4f6b)

### 解决方案
[微服务架构下分布式事务解决方案 —— 阿里GTS](https://url.cn/5q6IveA)
