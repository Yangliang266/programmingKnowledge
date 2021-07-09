## zookeeper

### 是什么

> 分布式协调组件，通过zookeeper协调在分布式下各个节点访问的顺序一致性的问题

### zookeeper设计的整体猜想



###  为什么用

> 分布式框架下的产物，集群节点的分布式一致性

#### 特点

##### 构建分布式集群：高可用 / 集群内部采用cp模型

- 高可用即**分区容错性（Partition tolerance）** 在网络异常情况下，系统仍能正常运作 --> leader选举

- cp模型即保证同一时刻同一请求的不同实例返回的结果相同，这要求数据具有强一致性  --> 数据同步

##### 为保证上述特性，集群节点有以下特点

1. 集群节点的分类

   > leader/follow/observer

2. 各集群节点的功能

   1. leader

      > 负责请求，事务的分发，具有总体调度的功能
      >
      > 数据的同步

   2. follow

      > 在leader出现异常时，负责leader的选举，半数选举
      >
      > 处理leader分发的请求
      >
      > 数据同步，半数确认原则

   3. obsever

      > 不进行leader的选举，数据同步半数确认原则
      >
      > 只是作为leader数据同步的节点和处理leader请求的分发（提升性能）

3. 分布式事务一致性

   > 2pc协议/3pc协议（分布式事务一致性，在跨节点时，保证事务的一致）





##### 数据一致性的保证：zab协议

ZAB 协议是为分布式协调服务ZooKeeper专门设计的一种支持崩溃恢复的一致性协议。基于该协议，ZooKeeper 实现了一种主从模式的系统架构来保持集群中各个副本之间的数据一致性。今天主要看看这个zab协议的工作原理。

**什么是ZAB协议**

话说在分布式系统中一般都要使用主从系统架构模型，指的是一台leader服务器负责外部客户端的写请求。然后其他的都是follower服务器。leader服务器将客户端的写操作数据同步到所有的follower节点中。

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/0ff41bd5ad6eddc4dae97672961710fb53663304.jpeg)

就这样，客户端发送来的写请求，全部给Leader，然后leader再转给Follower。这时候需要解决两个问题：

（1）leader服务器是如何把数据更新到所有的Follower的。

（2）Leader服务器突然间失效了，怎么办？

因此ZAB协议为了解决上面两个问题，设计了两种模式：

（1）消息广播模式：把数据更新到所有的Follower

（2）崩溃恢复模式：Leader发生崩溃时，如何恢复

OK。现在带着这两个问题，我们来详细的看一下：

**ZAB协议工作原理**

**1 消息广播模式：**

如果你了解过2PC协议的话，理解起来就简单很多了，消息广播的过程实际上是一个简化版本的二阶段提交过程。我们来看一下这个过程：

1. Leader收到请求之后，将它转换为一个proposal提议，并且为每个提议分配一个全局唯一递增的事务ID：zxid，然后把提议放入到一个FIFO的队列中，按照FIFO的策略发送给所有的Follower
2. Follower收到提议之后，以事务日志的形式写入到本地磁盘中，写入成功后返回ACK给Leader
3. Leader在收到超过半数的Follower的ACK之后，即可认为数据写入成功，就会发送commit命令给Follower告诉他们可以提交proposal了

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/v2-ae9a823d6576cf002aa22a3cc366be43_720w.jpg)

 **2 那么，Zookeeper如何进行Leader选举的(崩溃恢复)？**

Leader的选举可以分为两个方面，同时选举主要包含事务zxid和myid，节点主要包含LEADING\FOLLOWING\LOOKING3个状态。

1. 服务启动期间的选举
2. 服务运行期间的选举

**服务启动期间的选举**

1. 首先，每个节点都会对自己进行投票，然后把投票信息广播给集群中的其他节点
2. 节点接收到其他节点的投票信息，然后和自己的投票进行比较，首先zxid较大的优先，如果zxid相同那么则会去选择myid更大者，此时大家都是LOOKING的状态
3. 投票完成之后，开始统计投票信息，如果集群中过半的机器都选择了某个节点机器作为leader，那么选举结束
4. 最后，更新各个节点的状态，leader改为LEADING状态，follower改为FOLLOWING状态

**服务运行期间的选举**

如果开始选举出来的leader节点宕机了，那么运行期间就会重新进行leader的选举。

1. leader宕机之后，非observer节点都会把自己的状态修改为LOOKING状态，然后重新进入选举流程
2. 生成投票信息(myid,zxid)，同样，第一轮的投票大家都会把票投给自己，然后把投票信息广播出去
3. 接下来的流程和上面的选举是一样的，都会优先以zxid，然后选择myid，最后统计投票信息，修改节点状态，选举结束





##### zookeeper的数据结构

> zookeeper的内部的存储以kv的形式，以类似的树的结构进行分布排序

###### 存储节点的特性

- 节点的有序性(根据请求顺序排列)

  - 持久化节点 

  - 临时性节点（生命周期为一个会话）

    - 不允许有子节点

    > 先有父节点再有子节点，同级节点名字唯一

- 节点状态

  - cversion - 子节点版本
  - dataversion - 每次数据更新即更新版本号



##### zookeeper的监控watch

> 举例：dubbo的服务的注册与发现

- provider与zookeeper节点进行长连接保持通信
- consume订阅zookeeper节点，当provider节点发生变化时push给consume
  - zookeeper的监控watch，具有一次性特性，即订阅之后变化推送为一次
  - **循环注册订阅**，保证consume能随时接收provider节点的变化



##### zookeeper的应用场景

- 分布式锁 - 同一时间段处理一个请求，数据的一致性
- 全局id - 有序节点
- 配置中心
- 服务注册





