## 什么是kafka

> kafka不仅可以看作是消息中间件，也可以叫做消息引擎，或者是分布式实时流处理平台

### kafka的架构组成

![kafka](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/kafka.png)



#### 架构说明

##### 生产者 - producter

> 消息的发送者，在kafka中消息的发送是批量发送

##### broker

> 相当于消息收发的中介，生产者和消费者都需要和它建立连接，可以看作是一个服务

- topic

  > 生产者和消费者是通过topic关联起来的，是一个队列，可以理解为一组消息的集合（不同业务用途的消息），生产者和topic与消费者和topic都是多对多的关系（不建议）

  - partition

    > 一个topic可以划分为多个分区，为了解决数据的横向扩展和并发负载问题

  - replica

    > 副本数不能大于broker的数量，其存在的目的是为了解决kafka的高可用

##### 消费者 - consumer

> 在kafka中消息的获取是pull获取，

- 消费者组 - consumer group

  > 一个消费组中的消费者只能消费topic中的一个partition





##  为什么使用kafka

### kafka的特性

#### 消息中间件

> 其他消息中间可以做的事情它都可以做，



#### 数据集成+流计算



#### 大数据





