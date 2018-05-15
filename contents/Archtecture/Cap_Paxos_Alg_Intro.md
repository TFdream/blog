## CAP 理论
[CAP](https://zh.wikipedia.org/wiki/CAP%E5%AE%9A%E7%90%86)定理是由加州大学伯克利分校Eric Brewer教授提出来的，他指出WEB服务无法同时满足一下3个属性：
* 一致性(Consistency) ： 客户端知道一系列的操作都会同时发生(生效)
* 可用性(Availability) ： 每个操作都必须以可预期的响应结束
* 分区容错性(Partition tolerance) ： 即使出现单个组件无法可用,操作依然可以完成

根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项。

## 2PC & 3PC

## Paxos 算法

## Raft 算法

## 一致性哈希

## 康威定律
在康威的这篇文章中，最有名的一句话就是：
```
Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations. - Melvin Conway(1967)
```
中文直译大概的意思就是：设计系统的组织，其产生的设计等同于组织之内、组织之间的沟通结构。


## 参考资料
[从 CAP 理论到 Paxos 算法](http://blog.longjiazuo.com/archives/5369)
[多IDC的数据分布设计(二)](https://timyang.net/tag/paxos/)
