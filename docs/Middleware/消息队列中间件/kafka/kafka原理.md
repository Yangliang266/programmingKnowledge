### 1 生产者

#### 1.1 main线程

- 消息的定制比如，消费一条消息要多少钱
- key-value的序列化
- 分区的选择
  - 指定分区
  - 未指定分区，自定义分区器
  - 未指定分区，也未定义分区器
    - hash取模
    - 整数自增取模

#### 1.2 sender线程（发送消息）

- 消息的生产会添加到消息累加器中（currenthashmap），key代表topic队列，value代表具体的消息，最终唤醒sender线程发送



### 2 服务端

#### 2.1 怎么接受消息

##### 2.1.1 判定机制

> 正常工作的follow节点，同步成功（正常是指节点间的通信正常，可以正常同步） -> ack  -> product生产者

##### 2.1.2 服务端ack的类型

- 0 只关心消息的发送，不管消息是否发送成功
- 1 只要leader节点接受消息成功，即判定follow同步成功
- -1 / all 所有的follow节点刷盘成功，返回ack（如若leader出问题ack返回失败，retry设置为0，不重试，避免造成数据重复）

> 以上性能一次递减吞吐量递减，数据的健壮性递增

##### 2.1.3 leader选举

> 当leader在同步数据时出现问题，怎么解决，进行leader的重新选举

- 通过zk在所有的broker中选举一个controller，创建一个临时节点进行管理
  - 监听broker变化
  - 监听topic变化
  - 监听partition变化
  - 获取和管理broker，topic，partition的信息
  - 管理partition的主从信息

- leader从哪些节点中选举

  > 上述的正常工作的follow节点会放到一个ISR容器中（set集合）
  >
  > 正常情况下在controller的管理下从OSR进行leader选举（同步选举）
  >
  > 但是如果ISR中没有任何同步节点，需要从OSR中进行选举（不干净的选举）

  - AR

    > 所有的节点集合

  - ISR

    > 同步的节点集合

  - OSR

    > 未同步的节点集合

- 选举算法

  - PacifcA算法，最早原则，节点id最小

- 主从同步

  <img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/kafka%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5.png" alt="kafka主从同步" style="zoom:80%;" />

  - LEO

    > 下一个等待写入消息的offset

  - HW

    > ISR中最小的LEO

  - 流程

    - follow - fetch - leader - 发送数据 - follow
    - follow一次写入消息，更新LEO
    - leader更新HW

- 为什么不使用zookeeper进行选举

  - kafka有众多分区，如果出现大量分区失效，zk负载量大大增加
  - leader节点副本失效，产生大量watch事件
  - 只是利用zk，在broker中选举出一个controller节点，作为管理者



#### 2.2 接受消息后怎么处理

##### 2.2.1 partition分区的创建（可扩展）

> 分区的创建有助于数据的横向扩展，一个partition的消息是有序的，顺序写入，但是全局不一定有序

##### 2.2.2 replica副本的创建（高可用）

> 提高分区的可靠性，副本数不得超过broker数
>
> 一个topic分区的leader节点不会在同一broker，会尽可能错开在不同的broker中，这样避免产生多次选举

##### 2.2.3segement分段的创建（容量膨胀）

> 为了防止log不断追加导致文件过大，影响查找速率，一个partition被划分为多个segement

##### 2.2.4 数据文件

> 一个partition会有一套日志文件，被拆分成segement也以以下相同的文件形式出现

- index（偏移量索引文件）

  > 如果通过offset偏移量，就必须有一种快速检索消息的机制，即索引
  >
  > 记录偏移量offset和消息物理地址（log）的映射关系

  - 稀疏索引，写入的消息超过设定的大小建立一个索引

- log日志文件

  - 拆分条件
    - 日志文件大小
    - 消息的最大时间戳和当前时间的差值
    - index或者timeindex索引文件到了一定的大小

- timeindex（时间戳索引文件）

  > 时间戳和索引的关系

  - 基于时间切分日志
  - 基于时间清理消息

##### 2.2.5 消息的清理策略

- 开关
- 策略
  - delete 定时任务删除
    - 时间
    - 文件大小
  - compact 压紧
    - 把重复的key剔除





### 消费者

#### 消费者的消息记录

- consumer-offset文件（分区文件）
  - 消费者组中各个消费者的信息
  - 消费者组和各个partition的offset位移信息元数据
  - 消费者组中的一个消费者与partition的关系保存 - 根据偏移量继续消费
- 分区确定（50）
  - 消费者组的hash%50 - 哪个分区
- 新消费者的消费起点
  - earliest/lastest/none
- 分区文件的提交
  - 手动
    - 同步 - 更新消费组中的消费者对应分区的关系
    - 异步
  - 自动

#### 消费者的消费策略

##### 范围分配

> partition1,2 - consume1  /  partition3,4 - consumer2

##### 轮询分配

> partition1 - consume1 / partition2 - consume2 / partition3 - consume1 / partition4 - consume2 

##### 粘滞分配

> 带有随机性

##### 指定分配



#### 消费者分区重分配

> 如果topic的分区数或者消费者数量发生变化需要重分配

- 找出监督者

  > 每个broker都有用来管理offset和消费者的实例coordinator，挑选一个出来

- 所有消费者到coordinator， join group

- 挑选leader，告知所有消费者分区方案





### kafka快的原因

#### 顺序读写

> 磁盘的顺序io的读写速度要大于随机读的速度

#### 索引

#### 批量读写

#### 零拷贝

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/linkaob.png)

- 省去cpu拷贝
- DMA直接内存访问（拷贝）- 页缓存到socket



