> **基于curator实现分布式锁**

> **释放锁的逻辑**



#### zookeeper实现leader选举（集群下的leader选举)

> 在分布式计算中， leader election 是很重要的一个功能，这个选举过程是这样子的：指派一个进程作为组织者，将任务分发给各节点。在任务开始前，哪个节点都不知道谁是 leader 或者 coordinator。当选举算法开始执行后，每个节点最终会得到一个唯一的节点作为任务 leader。除此之外，选举还经常会发生在 leader 意外宕机的情况下，新的 leader 要被选举出来。

Curator 有两种选举 recipe（ Leader Latch 和 LeaderElection）

**1 leader latch**

参与选举的所有节点，会创建一个顺序节点，其中最小的节点会设置为 master 节点, 没抢到 Leader 的节点都监听前一个节点的删除事件，在前一个节点删除后进行重新抢主，当 master 节点手动调用 close（）方法或者 master节点挂了之后， 后续的子节点会抢占 master。

其中 spark 使用的就是这种方法

**2 leaderSelector**

LeaderSelector 和 Leader Latch 最的差别在于， leader可以释放领导权以后，还可以继续参与竞争



#### Zookeeper 数据的同步流程

zookeeper 通过三种不同的集群角色来组成整个高性能集群的在 zookeeper 中，客户端会随机连接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据，如果是写请求，那么请求会被转发给 leader 提交事务，然后 leader 会广播事务，只要有超过半数节点写入成功，那么写请求就会被提交（类 2PC 事务）

![img](C:\Users\xiaomi\AppData\Local\YNote\data\yangl546493589@163.com\48cb0d530075493bba77b4e6201d74e0\clipboard.png)

​	那么问题来了

1. 集群中的 leader 节点如何选举出来？

2. leader 节点崩溃以后，整个集群无法处理写请求，如何快速从其他节点里面选举出新的 leader 呢？

3. leader 节点和各个 follower 节点的数据一致性如何保证

**1 ZAB 协议**

ZAB（Zookeeper Atomic Broadcast） 协议是为分布式协调服务 ZooKeeper 专门设计的一种支持崩溃恢复的原子广播协议。在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议， ZooKeeper 实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性

**2 zab 协议介绍**

ZAB 协议包含两种基本模式，分别是

1. **崩溃恢复**

2. **原子广播**

当整个集群在启动时，或者当 leader 节点出现网络中断、崩溃等情况时， ZAB 协议就会进入恢复模式并选举产生新的 Leader，当 leader 服务器选举出来后，并且集群中有过半的机器和该 leader 节点完成数据同步后（同步指的是数据同步，用来保证集群中过半的机器能够和 leader 服务器的数据状态保持一致）， ZAB 协议就会退出恢复模式。当集群中已经有过半的 Follower 节点完成了和 Leader 状态同步以后，那么整个集群就进入了消息广播模式。这个时候，在 Leader 节点正常工作时，启动一台新的服务器加入到集群，那这个服务器会直接进入数据恢复模式，和leader 节点进行数据同步。同步完成后即可正常对外提供非事务请求的处理。

需要注意的是： leader 节点可以处理事务请求和非事务请求， follower 节点只能处理非事务请求，如果 follower 节点接收到非事务请求，会把这个请求转发给 Leader 服务器

**3 消息广播的实现原理**

如果大家了解分布式事务的 2pc 和 3pc 协议的话（不了解也没关系，我们后面会讲），消息广播的过程实际上是一个简化版本的二阶段提交过程

1. leader 接收到消息请求后，将消息赋予一个全局唯一的64 位自增 id，叫： zxid，通过 zxid 的大小比较既可以实现因果有序这个特征

2. leader 为每个 follower 准备了一个 FIFO 队列（通过 TCP协议来实现，以实现了全局有序这一个特点）将带有 zxid的消息作为一个提案（proposal）分发给所有的 follower

3. 当 follower 接收到 proposal，先把 proposal 写到磁盘，写入成功以后再向 leader 回复一个 ack

4. 当 leader 接收到合法数量（超过半数节点）的 ACK 后，leader 就会向这些 follower 发送 commit 命令，同时会在本地执行该消息

5. 当 follower 收到消息的 commit 命令以后，会提交该消息

![img](C:\Users\xiaomi\AppData\Local\YNote\data\yangl546493589@163.com\7c9340229e53430b80ef4746419350c6\clipboard.png)

ps: 和完整的 2pc 事务不一样的地方在于， zab 协议不能终止事务， follower 节点要么 ACK 给 leader，要么抛弃leader，只需要保证过半数的节点响应这个消息并提交了即可，虽然在某一个时刻 follower 节点和 leader 节点的状态会不一致，但是也是这个特性提升了集群的整体性能。 当然这种数据不一致的问题， zab 协议提供了一种恢复模式来进行数据恢复

这里需要注意的是：

leader 的投票过程，不需要 Observer 的 ack，也就是Observer 不需要参与投票过程，但是 Observer 必须要同步 Leader 的数据从而在处理请求的时候保证数据的一致性

**4 崩溃恢复的实现原理**

前面我们已经清楚了 ZAB 协议中的消息广播过程， ZAB 协议的这个基于原子广播协议的消息广播过程，在正常情况下是没有任何问题的，但是一旦 Leader 节点崩溃，或者由于网络问题导致 Leader 服务器失去了过半的 Follower 节点的联系（leader 失去与过半 follower 节点联系，可能是leader 节点和 follower 节点之间产生了网络分区，那么此时的 leader 不再是合法的 leader 了） ，那么就会进入到崩溃恢复模式。 崩溃恢复状态下 zab 协议需要做两件事

1. **选举出新的 leader**

2. **数据同步**

前面在讲解消息广播时，知道 ZAB 协议的消息广播机制是简化版本的 2PC 协议，这种协议只需要集群中过半的节点响应提交即可。但是它无法处理 Leader 服务器崩溃带来的数据不一致问题。 因此在 ZAB 协议中添加了一个“崩溃恢复模式”来解决这个问题。那么 ZAB 协议中的崩溃恢复需要保证，如果一个事务Proposal 在一台机器上被处理成功， 那么这个事务应该在所有机器上都被处理成功，哪怕是出现故障。 为了达到这个目的，我们先来设想一下，在 zookeeper 中会有哪些场景导致数据不一致性，以及针对这个场景， zab 协议中的崩溃恢复应该怎么处理

> 已经被处理的消息不能丢

> 被丢弃的消息不能再次出现

**5 关于 ZXID**

前面一直提到 zxid，也就是事务 id，那么这个 id 具体起什么作用，以及这个 id 是如何生成的， 简单给大家解释下为了保证事务的顺序一致性， zookeeper 采用了递增的事务 id 号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了 zxid。实现中 zxid 是一个 64 位的数字，它高 32 位是 epoch（ZAB 协议通过 epoch 编号来区分 Leader 周期变化的策略） 用来标识 leader 关系是否改变，每次一个 leader 被选出来，它都会有一个新的epoch=（原来的 epoch+1） ，标识当前属于那个 leader 的统治时期。低 32 位用于递增计数。

epoch：可以理解为当前集群所处的年代或者周期，每个leader 就像皇帝，都有自己的年号，所以每次改朝换代， leader 变更之后，都会在前一个年代的基础上加1。这样就算旧的 leader 崩溃恢复之后，也没有人听他的了，因为 follower 只听从当前年代的 leader 的命令。 