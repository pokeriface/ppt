title: consensus
speaker: liujun
transition: move
theme: light
files: /js/demo.js,/css/demo.css
date: 2017年3月25日

[slide]

# 分布式环境下一致性问题的解决方案
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
* 数据库事务的一致性：事务的执行必须是从一个一致性状态进入到另一个一致性状态。例如触发器级联等约束。 {:&.moveIn}
* 分布式系统的一致性：指数据在多个副本之间是否能够保持一致的特性。也即数据在一致性状态更新后，仍然处于一致性的状态。

[slide]
-----
###不同的分布式一致性级别
* 强一致性：在任意时刻，读请求总能读到最新的数据。 {:&.moveIn}
* 弱一致性：不承诺立即读到最新的数据，但尽可能在某个时间后，达到一致性状态。
* 最终一致性：是弱一致性的特例。系统承诺在一定时间内，达到一致性状态。最为推崇的模型，它存在五种变种。
 * 因果一致性：进程A更新完数据后通知进程B，B可以读到最新的数据。 {:&.moveIn}
 * 读己已提交：更新的进程可以读到最新的数据。
 * 会话一致性：在同一个Session中保证读己已提交。
 * 单调读一致性：系统保证对于任意单个进程的读操作，不会读到更旧的。
 * 单调写一致性：保证同一个进程的写操作被顺序执行。

[slide]
-----
### 分布式系统的相关理论
* CAP定理：分布式系统不可能同时满足一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）这三个基本需求，最多只能满足其二。 {:&.moveIn}
* BASE理论：基本可用（Basically Available）、软状态（Soft state）和最终一致性（Eventually consistent）。既然无法做到强一致性，那么每个应用都可以根据自己的业务特点进行折中，使之达到最终一致性。

[slide]
### 问题的描述
-----
* 目标：为了构建一个_高可用、具备一定一致性_的系统。 {:&.moveIn}
* 挑战：分布式系统采用异步通信模式，消息可能会丢失、可能会乱序、可能会重复。通信节点可能会宕机。
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

1. 主备同步出现网络抖动，最大可用模式与最大保护模式的选择（CA的选择）。 {:&.moveIn}
2. HA模块与主server出现网络抖动。切还是不切？这是一个问题。
3. HA模块本身是个单点，它的高可用由谁来保证？
4. 共性问题。挂掉一个，瞬间就成了单点。
5. 主备模式不能很好的避免网络抖动。 

[slide]
###一些一致性算法
|算法|实际应用|
|---- |--------------------:|
|两阶段提交协议|一般用于分布式事务的提交。|
|三阶段提交协议|基于两阶段提交，解决了阻塞的问题。|
|Basic Paxos|MySQL 5.7的Group Repication模块，微信开源PhxSQL。|
|Raft|TiDB等。|
|Multi Paxos|OceanBase数据库|

[slide]
## 二、Paxos 一致性算法
-----
![Alt text](/img/leslie.jpg "Optional title")


* 问题的阐述 {:&.moveIn}
* Paxos协议的推导
* 完整的Paxos协议
* 总结

[slide]
### 问题的阐述
* 一个变量value，分布在N个进程中。每个进程都尝试修改自身value的值，它们企图各不相同，但最终所有的进程会对value的某个值达成一致。 {:&.moveIn}
* 前提只有一个，不存在拜占庭将军的问题。一致性算法的要求。
 * value达成一致的值必须是某个进程提出的。 {:&.moveIn}
 * 一旦value达成了一致，那么value必须不能被更改。也叫安全性。
 * 总能达成一致，不能无休止的进行。也称活性。


[slide]
### Paxos协议的推导
* 达成一致的条件：基于多数派的协议，过半原则。在Paxos中，这个多数派的投票集合叫做法定集合，只要确保了集合中每个元素只投一次票，则保证了不会达成两个结果。 
* 不同的成员负责提出提案，基于先到先得，每人投票一次，进行投票。？？
 * 有了提案号（proposer_id），能够对提案号做出区分，接收提案号更大的提案。问题得到部分解决。

![Alt text](/img/split_vote.jpg "Optional title")

[slide]
### Paxos协议的推导
* 仍然存在的问题，无法预知未来会不会有更大的提案号过来，还是没办法保证活性。
 * 让投票者能够拒绝提案号更大的提案？不可行。 {:&.moveIn}
 * 让投票者能够拒绝提案号更小的提案？分裂成两个阶段是一个思路。第一阶段用于收集信息，并投票人作出承诺。第二阶段选举人得知自己过半了的话，则宣布当选。
 * 限制后收到的提案，使其提案的内容与前者相同？！

[slide]
### Paxos协议的推导
* 总结一下，目前在功能上做区分，可以划分为两种角色。Proposer和Acceptor。 {:&.moveIn}
* 目前划分为两种阶段，Prepare阶段和Accept阶段。
 * Prepare阶段：提议者Proposer向所有Acceptor广播预提案，需带上这次的提案号proposer_id。接受者Acceptor收到消息，更新本地的current_max_proposer_id = MAX(current_max_proposer_id, proposer_id)，承诺Proposer不答应比current_max_proposer_id小的提案。 {:&.moveIn}
 * Accept阶段：提议者Proposer正式广播自己的提案，与之前的proposer_id相同。接受者Acceptor收到消息，再次更新current_max_proposer_id，并决定是否给这个Proposer投票。
* 仍然存在问题！假设某次提案只有PA参与，提案信息(proposerid_a, valuea)PA显然过半选片当选，这时PB拥有更大的proposer_id，显然PB当选，这时就导致了value在整个选举中的不一致。

[slide]
### 需要一个什么策略？
* 如果一个提案[M0, V0]被选定，那么提案编号大于M0的Proposer所产生的提案，其value均为V0。 {:&.moveIn}
* 利用手头收集到的信息，Prepare阶段可以得到所有Acceptor节点承诺过的提案，从中选出这个V0。使得Accept阶段提出的提案(Mn,Vn)中Vn=V0。
* 利用第二数学归纳法来反推这个策略。
 * 假设编号M0到M(n-1)之间的提案，其value值都是V0，证明编号为Mn的提案的Value值也为V0。
 * 得出策略结论：对于任意的Mn和Vn，如果提案[Mn, Vn]被提出，集合S必须满足两个条件之一。 S没有批准过小于Mn的提案。或者S中所有Acceptor批准的编号小于Mn的提案编号最大的提案的值，它的值等于V0。

[slide]
### 完整的Paxos协议
-----
**Prepare阶段**
* Proposer节点生成新的提案编号pid，然后向某个Accept集合成员发送Prepare请求。 {:&.moveIn}
* Acceptor节点可能在任意时刻收到Prepare请求。如果收到的P大于Acceptor已经收到过的提案编号max_id，则更新自己的最新提案编号max_id，然后回应Proposer承诺不再批准任何小于max_id的提案。反之，则不回应。

**Accept阶段**

* Proposer节点收到过半的Acceptor承诺，则产生新的提案(pid, value_n)。其中value_n为所有在prepare阶段收到的响应中提案编号最大的提案编号所对应的value，如果半数以上都没有批准任何提案，则说明半数Acceptor批准的都是自己的value，则value_n可以是任意值。 {:&.moveIn}
* 不违背Acceptor的现有承诺下，相应任意Accept请求。

[slide]
### 总结
* 为分布式下的一致性提出了完整的证明，有很强的理论指导。 {:&.moveIn}
* 基于Paxos产生了很多变种的一致性算法。
* 抽象。
* 难以实现。

[slide]
## 三、Raft 一致性算法
1. Raft概述 {:&.moveIn}
2. Raft算法基础
3. leader选举
4. 日志同步
5. 正确性
6. 总结

[slide]
### Raft概述
* 既然有了Paxos，为什么还会有Raft？ {:&.moveIn}
 * Paxos是一种理论上的一致性算法，保证了正确性。但又太过抽象，让人难以理解。Raft作者提出一种从流程上简化了的一致性算法。 {:&.moveIn}
 * Paxos论文本身提供了理论的思路，直接用于工程更加复杂而且未必有很好的性能。Raft算法本身就已经模块化，关键流程上给出了伪代码级别的过程。
 * 总结：Paxos 算法的描述与实际实现之间存在巨大的鸿沟，最终的系统往往建立在一个没有被证明的算法之上。
* Raft算法的特点？
 * 强化了leader这一概念。 {:&.moveIn}
 * 分为 leader选举与日志同步两个模块。
 * 支持了集群配置的动态变更。

[slide]
### Raft算法基础
* 整体流程
 * 选举出一个leader，如果有过半的server投票给该server，则当选为leader。
 * client发送信息到leader，由leader单向的向所有server同步log，过半的commit该条log。然后无限重试同步给其他的server。
* 一些关键词
 * term号，在Raft算法中扮演逻辑时钟的角色。以及current term。
 * log以及log index。
 * commit index。
 * AppendEntriesRPC和RequestsVoteRPC。
* server角色的划分
![Alt text](/img/server_roles.png)

[slide]
### Raft算法基础
-----------
|特性|描述|
|---- |--------------------:|
|选举安全原则|一个任期最多选出一个leader|
|领导人只增加原则|领导人不会删除覆盖自己的日志|
|日志匹配原则|如果日志有相同日志号，且对应的term相同，则认为从头到此的日志完全相同|
|领导人完全原则|如果日志在一个任期内被提交，则一定会出现在任期更大的领导人中|
|状态机安全原则|如果某条日志应用到了状态机中，则其他服务器不会在该位置应用其他日志|
* Raft在任意时刻保证这些原则都成立。
* 保证了这些原则的成立，也就变相的保证了一致性算法的安全性。

[slide]
### Raft算法基础
-----------
![Alt text](/img/replicate_machine.png)

经典复制状态机
* 安全性 {:&.moveIn}
* 高可用
* 不依赖时序
* 允许少量慢节点

[slide]
###Leader选举
* 总体上使用心跳来触发选举。 {:&.moveIn}
* 初始化状态为所有server都是follower，当超过自定义的选举时间，则该server会从follower状态变成condidate。自增本地current term，并发起RequestsVoteRPC，号召大家给自己投票。
* 接收到candidate发起的RequestsVoteRPC时，server做以下逻辑的判断。
 * 如果接收到的term号小于自己的current term号，则返回false。
 * 如果没有给其他人投过票或者已经给他投过票了，并且候选人的最新log index与自己的log index相等，则给它投票。
* 上述动作会产生三种结果。
 * 赢得了选举，当选leader。立即向所有server告知当选leader，防止再次选举。
 * 收到了其他的RequestsVoteRPC，如果收到的current term大于自己的，则退化为follower，否则拒绝并继续选举。
 * 没有server赢得多数派，超时重试。

[slide]
**如何避免投票分裂**
* 基于优先级的算法。
* 基于随机时间退避的算法。随机的选举超时时间，保证只有一个server会率先超时，保证选票集中。 {:&.moveIn}

[slide]
### 日志同步
* 只支持单向的日志流，由leader向所有server同步。过半则commit。 {:&.moveIn}
* 当follower接收到AppendEntriesRPC时，要做以及逻辑判断。
 * 如果 term < currentTerm，返回 false。 {:&.moveIn}
 * 如果在prevLogIndex处的日志的任期号与prevLogTerm不匹配时，返回 false。
 * 如果一条已经存在的日志与新的冲突（index 相同但是任期号 term 不同），则删除已经存在的日志和它之后所有的日志。
 * 添加任何在已有的日志中不存在的条目。
 * 如果leaderCommit > commitIndex，将commitIndex设置为leaderCommit和最新日志条目索引号中较小的一个。

[slide]
### 正确性
* 虽然被划分为两个模块，但相互作用保证正确性。利用日志的连续性，保证选举出来的leader必然拥有全部的日志。 {:&.moveIn}
* 领导人完全原则的证明。假设存在leader T任期，U是T接下来的一个任期，且与T所对应的leader不同，推出与我们的已知矛盾。
 * 前提：leader U不存在某条已提交的日志。 {:&.moveIn}
 * leader U被过半选出来，而这条日志已提交，同样已被过半的follower同意。则必定有一台follower同时满足给U投票，且拥有这条已提交的日志。这台follower是推出矛盾的关键。
 * 该投票者若在接收日志后，收到投票请求。不满足投票中投票条件，矛盾。
 * 投票者若在投票后，收到接收日志同步请求，term号变大，拒绝日志同步，矛盾。

[slide]
### 与客户端的交互
* 随机连接一台，如果是leader则连接成功，连接的为follower则返回客户端它的最新的term。
* 给每个指令一个唯一的id，这样客户端超时退出后，可以根据这个id检验是否执行成功，没成功则leader重新执行。

[slide]
### 总结
* 提出了一种可理解性更高的一致性算法。 {:&.moveIn}
* 使得工程级的方案更容易落地。
* 工程上的缺点，要求日志是连续的。不允许有空洞。

[slide]
## 四、分布式协调系统——ZooKeeper
----
1. 概述
2. 数据模型
3. 基本命令使用
4. 典型的应用场景
5. 最佳实践——依赖于ZooKeeper的选举算法实现
6. 总结

[slide]
### 概述
* Chubby的开源实现。用来解决分布式数据的一致性问题。 {:&.moveIn}
* 特性：
 * 顺序一致性。 {:&.moveIn}
 * 原子性。
 * 单一视图特性
 * 可靠性
 * 相对实时，提供最终一致性。
* 设计目标：
 * 简单。 {:&.moveIn}
 * 集群式部署。
 * 顺序访问。
 * 高性能。

[slide]
### 数据模型及特性
----
![Alt Text](/img/zknamespace.jpg)
* 提供类似于文件系统的数据模型。但没有目录与文件的概念，统一为ZNode，可以理解为目录和文件的结合。 {:&.moveIn}
* 提供了监视（Watcher）功能，监视某些事件的发生。

[slide]
###基本命令使用
----
|命令 | 参数 | 简述|
|---- |:------:|-------:|
|create path| -e -s | 创建ZNode，分为是否为临时和是否为顺序节点的属性|
|ls path||查看path下的孩子节点|
|get path||查看节点的数据内容以及属性|
|set path data|可指定固定版本|设置节点|
|delete path|可指定固定版本|删除节点|

[slide]
###典型的应用场景
----
1. 分布式的配置中心
2. 分布式命名服务
3. 分布式协调通知
4. 分布式锁协议
5. Master选举
6. 分布式队列
7. ......

[slide]
### 最佳实践——依赖于ZooKeeper的选举算法实现
![Alt Text](/img/election_based_zookeeper.jpg)

[slide]
### 总结
* 提出一种数据模型，极大的简化了分布式系统的开发。 {:&.moveIn}
* 使用以及部署简单、灵活，比较足够轻量。
* 提出ZAB算法（Paxos的变种）来保证一致性。

[slide]
# Q & A

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
9. http://www.infoq.com/cn/articles/raft-paper
10. http://www.allprogrammingtutorials.com/tutorials/leader-election-using-apache-zookeeper.php
11. http://www.allthingsdistributed.com/2008/12/eventually_consistent.html
12. https://zhihu.com/question/19787937/answer/82340987
