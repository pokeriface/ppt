title: consensus
speaker: liujun
transition: move
theme: light
files: /js/demo.js,/css/demo.css
date: 2017年3月25日

[slide]

# 分布式环境下的一致性问题
## 演讲者：Liujun

[slide]
# 大纲
1. 一致性简介 {:&.moveIn}
2. Paxos 一致性算法
3. Raft 一致性算法
4. 分布式协调系统——ZooKeeper

[slide]
## 一、 一致性简介
------
* 数据库事务的一致性：事务的执行必须是从一个一致性状态进入到另一个一致性状态。 {:&.moveIn}
* 分布式系统的一致性：指数据在多个副本之间是否能够保持一致的特性。也即数据在一致性状态更新后，仍然处于一致性的状态。

[slide]
-----
###不同的分布式一致性级别
* 强一致性：在任意时刻，所有节点的数据是一致的。 {:&.moveIn}
* 弱一致性：不承诺立即读到最新的数据，但尽可能在某个时间后，达到一致性状态。
* 最终一致性：是弱一致性的特例。系统承诺在一定时间内，达到一致性状态。最为推崇的模型，它存在五种变种。
 * 因果一致性：进程A更新完数据后通知进程B，B可以读到最新的数据。 {:&.moveIn}
 * 读己已提交：更新的进程可以读到最新的数据。
 * 会话一致性：在同一个Session中保证读己已提交。
 * 单调读一致性：系统保证对于任意单个进程的读操作，不会读到更旧的。
 * 单调写一致性：保证同一个进程的写操作被顺序执行。

[slide]
-----
###分布式系统的相关理论
* CAP定理：分布式系统不可能同时满足一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）这三个基本需求，最多只能满足其二。 {:&.moveIn}
* BASE理论：基本可用（Basically Available）、软状态（Soft state）和最终一致性（Eventually consistent）。既然无法做到强一致性，那么每个应用都可以根据自己的业务特点进行折中，使之达到最终一致性。

[slide]
###问题的描述
-----
* 目标：为了构建一个_高可用、具备一定一致性_的系统。 {:&.moveIn}
* 挑战：分布式系统采用异步通信模式，消息可能会丢失、可能会乱序。通信节点可能会宕机。
* 换句话说，我们的目标是在不可靠的服务器节点、不可靠的通信模式之上，构建一个高可用的分布式系统。

[slide]
###案例分析
-----
* 场景：MySQL的Master-Slave同步模式，分为同步、异步和半同步模式。 为了简化问题的说明，配置为一主一从同步的方式。

![Alt text](/img/master_slave.jpg "Optional title")
-----
* 当出现问题时，需要人工的介入进行主备的切换。 {:&.moveIn}


[slide]
###带HA模块的方案
-----
![Alt text](/img/ha_frame.png "Optional title")

1. 主备同步出现网络抖动，最大可用模式与最大保护模式的选择（CAP的选择）。 {:&.moveIn}
2. HA模块与主server出现网络抖动。切还是不切？这是一个问题。
3. HA模块本身是个单点，它的高可用由谁来保证？
4. 共性问题。挂掉一个，瞬间就成了单点。
5. 主备模式不能很好的避免网络抖动。 

[slide]
###一些一致性算法
* 2PC
* 3PC
* Paxos。MySQL Enterprise Edition，微信开源PhxSQL。
* Raft。TiDB等。
* Multi Paxos。OceanBase数据库。

[slide]
## 二、Paxos 一致性算法
![Alt text](/img/leslie.jpg "Optional title")

* Paxos概述
* 

[slide]
## 三、Raft 一致性算法
* Raft概述
* leader选举
* 日志同步
* 正确性
* 总结

[slide]
### Raft概述
* 既然有了Paxos，为什么还会有Raft？
 * Paxos是一种理论上的一致性算法，保证了正确性。但又太过抽象，让人难以理解。Raft作者提出一种从流程上简化了的一致性算法。
 * Paxos在工程上难以实现，也是因为太偏理论。Raft算法本身就已经模块化，关键流程上给出了伪代码级别的过程。
* Raft算法的特点？
 * 强化了leader这一概念。
 * 分为 leader选举与日志同步两个模块。
 * 支持了集群配置的动态变更。

[slide]
### Raft算法基础
* 整体流程
 * 选举出一个leader，如果有过半的server投票给该server，则当选为leader。
 * client发送信息到leader，由leader单向的向所有server同步log，过半的commit该条log。然后无限重试同步给其他的server。
* 模块的划分
 * leader选举。对应于RequestVote RPC。
 * 日志同步。对应于AppendEntries RPC。
* server角色的划分
![Alt text](/img/server_roles.png)

[slide]
###Leader选举
* 总体上使用心跳来触发选举。
* 

[slide]
### 总结
* 提出了一种可理解性更高的一致性算法。
* 使得工程级的方案更容易落地。
* 工程上的缺点，要求日志是连续的。不允许有空洞。


[slide]
## 四、分布式协调系统——ZooKeeper
----
1. 数据模型
2. 基本命令使用
3. 典型的应用场景
4. 最佳实践——Kafa中的ZooKeeper使用

[slide]
### 数据模型

[slide]
###基本命令使用

[slide]
###典型的应用场景
----
1. 分布式的配置中心
2. 分布式命名服务
3. 分布式锁协议
4. Master选举
5. ...

[slide]
###最佳实践——Kafka中的ZooKeeper使用
_Kafka的架构图_

[slide]
##Ref
1. [MySQL Enterprise Edition](https://www.mysql.com/products/enterprise/high_availability.html)
2. [微信开源PhxSQL：高可用、强一致的MySQL集群](http://djt.qq.com/article/view/1489)
3. http://hedengcheng.com/?p=892
4. 《Paxos Made Simple》
5. 《In Search of an Understandable Consensus Algorithm》
6. 《从Paxos到ZooKeeper：分布式一致性原理与实践》
7. http://zookeeper.apache.org/doc/r3.4.9/
8. http://mp.weixin.qq.com/s?__biz=MzI4NDMyNTU2Mw==&mid=2247483695&idx=1&sn=91ea422913fc62579e020e941d1d059e#rd
