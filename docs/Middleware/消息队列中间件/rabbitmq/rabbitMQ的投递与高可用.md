## rabbitmq高可用

### 1 rabbitmq集群

> 集群节点之间分为内存结点和磁盘节点
>
> 内存结点负责读写数据交互
>
> 磁盘节点负责备份

#### 2 rabbitmq普通集群

> 普通集群节点之间的数据同步只是元数据的同步（vhost的定义，队列和交换机的绑定关系）

- 产生的问题 - 如果内存节点故障则数据丢失

#### 3 rabbitmq镜像集群

> 镜像集群节点之间的数据同步不仅仅是元数据的同步，具体的消息也会被同步到磁盘节点

- 产生的问题 - 如果想进行数据的横向扩展？

  分片思想

##### 3.1 集群的高可用

<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/rabbitmq%E9%AB%98%E5%8F%AF%E7%94%A8.png" alt="rabbitmq高可用" style="zoom: 80%;" />

###### 3.1.1 如果存在多个内存节点生产者和消费者该如何进行节点的选择

- 对多个节点进行负载HAproxy
- 为了高可用也需要对负载进行集群的搭建
- 客户端选择哪个负载？
- keepalived
  - 虚拟路由冗余协议VRRP
  - 监控HAproxy节点，发生异常进行踢出
  - 可以部署多个服务，只有一个选举出来的master
  - keepalived master节点对外提供虚拟ip，供应用端连接，keepalived节点谁抢占到ip，就由谁对外提供网络服务

###### 3.1.2 HAproxy+keepalived+rabbitmq架构

<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/rabbitmq-keepalived-HAproxy.png" alt="rabbitmq-keepalived-HAproxy" style="zoom: 80%;" />



### rabbitmq的不足

RabbitMQ是非常流行的消息中间件，大家都知道通过集群能够增大它的吞吐量，那么针对单个队列，集群能增大他的吞吐量吗？如果不能，我们要怎么做呢？

答案是**集群并不能的增加单个队列的吞吐量**，这是因为RabbitMQ的普通集群只是共享元数据信息，达到将整个集群规模的队列扩大以增加吞吐量的目的。普通集群甚至不能保证消息数据的高可用，任意一个broker宕机，都会导致这个broker上的队列不可用。

而镜像队列也仅仅只是保证了实现镜像复制的队列的高可用。消费者并不能并发消费复制出来的队列。

那么RabbitMQ是否也能提高类似Kafka的topic分区的机制，来加大单个主题队列的吞吐量呢？

通过使用 RabbitMQ Sharding 插件、Consistent-hash Sharding Exchange 来更加灵活地动态均衡队列压力，可以更从容地达到百万并发的性能。