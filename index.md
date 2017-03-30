title: consensus
speaker: liujun
transition: move
theme: light
files: /js/demo.js,/css/demo.css
date: 2017年3月25日

[slide]

# 分布式一致性
## 演讲者：liujun

[slide]
# 大纲
1. 问题的提出 {:&.moveIn}
2. Paxos 一致性算法
3. Raft 一致性算法
4. 分布式协调系统——ZooKeeper

[slide]
## 一、 问题的提出
------


> 目标：
>     为了构建一个_高可用、强一致_的系统。 

[slide]
###传统的解决方案
-----

![Alt text](/img/master_slave.jpg "Optional title")
-----
* 当出现问题时，需要人工的介入进行主备的切换。 {:&.moveIn}



[slide]
###带HA模块的方案
-----
![Alt text](/img/ha_frame.png "Optional title")


+ 主备同步出现网络抖动，最大可用模式与最大保护模式的选择（CAP的选择）。 {:&.moveIn}
+ HA模块与主server出现网络抖动。切还是不切？这是一个问题。
+ HA模块本身是个单点，它的高可用由谁来保证？
+ 共性问题。挂掉一个，瞬间就成了单点。
+ 主备模式不能很好的避免网络抖动。

[slide]
## 二、Paxos 一致性算法

[slide]
## 三、Raft 一致性算法
----

```javascript
alert('nodeppt');
```

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
1. http://hedengcheng.com/?p=892
2. 《Paxos Made Simple》
3. 《In Search of an Understandable Consensus Algorithm》
4. 《从Paxos到ZooKeeper：分布式一致性原理与实践》
5. http://zookeeper.apache.org/doc/r3.4.9/
