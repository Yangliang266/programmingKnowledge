### rocketmq组成

#### product

#### topic

#### NameServer

#### brokerserver

#### message queue

- topic
- broker
- queueid

#### consumer





---



### rocketmq执行流程

![rocketmq_product_ack](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/rocketmq_product_ack.png)



#### 1 生产者发送消息

##### 1.1 数据发送时队列的选择（负载）

- 根据hash获取，轮询发送（默认）

- 随机获取

- 自定义 -> 选择队列

  ```java
  SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
      public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
          Integer id = (Integer) arg;
          int index = id % mqs.size();
          return mqs.get(index);
      }
  }, i);
  ```

  

##### 1.2 数据发送到返回状态流程

##### 1.2.1 消息的发送方式

- 成功
  
  - 同步
    - 原理：同步发送是指消息发送方发出数据后，会在收到接收方发回响应之后才发下一个数据包的通讯方式。
    - 场景：此种方式应用场景非常广泛，例如重要通知邮件、报名短信通知、营销短信系统等。
    - 类似推拉的形式 `发送 ->同步返回 ->发送 ->同步返回`
  - 异步
    - 原理：异步发送是指发送方发出数据后，不等接收方发回响应，接着发送下个数据包的通讯方式。 MQ 的异步发送，需要用户实现异步发送回调接口（SendCallback）。消息发送方在发送了一条消息后，不需要等待服务器响应即可返回，进行第二条消息发送。发送方通过回调接口接收服务器响应，并对响应结果进行处理。
    - 场景：异步发送一般用于链路耗时较长，对响应时间较为敏感的业务场景，例如用户视频上传后通知启动转码服务，转码完成后通知推送转码结果等。 耗时比较长的 可以不需要同步返回给用户的
  - oneway
  - 原理：单向（Oneway）发送特点为发送方只负责发送消息，不等待服务器回应且没有回调函数触发，即只发送请求不等待应答。 此方式发送消息的过程耗时非常短，一般在微秒级别。
    - 场景：适用于某些耗时非常短，但对可靠性要求并不高的场景，例如日志收集。
  
- 失败
  
  - 重试机制，最多重试2次
  - 如果同步模式发送失败，则轮转到下一个Broker，如果异步模式发送失败，则只会在当前Broker进行重试。这个方法的总耗时时间不超过sendMsgTimeout设置的值，默认10s。
  - 如果本身向broker发送消息产生超时异常，就不会再重试。
  
  > 以上策略也是在一定程度上保证了消息可以发送成功。如果业务对消息可靠性要求比较高，建议应用增加相应的重试逻辑：比如调用send同步方法发送失败时，则尝试将消息存储到db，然后由后台线程定时重试，确保消息一定到达Broker。



##### 1.2.2 状态码SendResult

- **SEND_OK**

消息发送成功。要注意的是消息发送成功也不意味着它是可靠的。要确保不会丢失任何消息，还应启用同步Master服务器或同步刷盘，即SYNC_MASTER或SYNC_FLUSH。

- **FLUSH_DISK_TIMEOUT**

消息发送成功但是服务器刷盘超时。此时消息已经进入服务器队列（内存），只有服务器宕机，消息才会丢失。消息存储配置参数中可以设置刷盘方式和同步刷盘时间长度，如果Broker服务器设置了刷盘方式为同步刷盘，即FlushDiskType=SYNC_FLUSH（默认为异步刷盘方式），当Broker服务器未在同步刷盘时间内（默认为5s）完成刷盘，则将返回该状态——刷盘超时。

- **FLUSH_SLAVE_TIMEOUT**

消息发送成功，但是服务器同步到Slave时超时。此时消息已经进入服务器队列，只有服务器宕机，消息才会丢失。如果Broker服务器的角色是同步Master，即SYNC_MASTER（默认是异步Master即ASYNC_MASTER），并且从Broker服务器未在同步刷盘时间（默认为5秒）内完成与主服务器的同步，则将返回该状态——数据同步到Slave服务器超时。

- **SLAVE_NOT_AVAILABLE**

消息发送成功，但是此时Slave不可用。如果Broker服务器的角色是同步Master，即SYNC_MASTER（默认是异步Master服务器即ASYNC_MASTER），但没有配置slave Broker服务器，则将返回该状态——无Slave服务器可用。



##### 1.3 事务处理

> RocketMQ事务消息（Transactional Message）是指应用本地事务和发送消息操作可以被定义到全局事务中，要么同时成功，要么同时失败。RocketMQ的事务消息提供类似 X/Open XA 的分布事务功能，通过事务消息能达到分布式事务的最终一致。

Apache RocketMQ在4.3.0版中已经支持分布式事务消息，这里RocketMQ采用了2PC的思想来实现了提交事务消息，同时增加一个补偿逻辑来处理二阶段超时或者失败的消息，如下图所示。

![rocketmq_design_10](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/rocketmq_design_10.png)

##### 1.3.1 RocketMQ事务消息流程概要

上图说明了事务消息的大致方案，其中分为两个流程：正常事务消息的发送及提交、事务消息的补偿流程。

1.事务消息发送及提交：

(1) 发送消息（half消息）。

(2) 服务端响应消息写入结果。

(3) 根据发送结果执行本地事务（如果写入失败，此时half消息对业务不可见，本地逻辑不执行）。

(4) 根据本地事务状态执行Commit或者Rollback（Commit操作生成消息索引，消息对消费者可见）

2.补偿流程：

(1) 对没有Commit/Rollback的事务消息（pending状态的消息），从服务端发起一次“回查”

(2) Producer收到回查消息，检查回查消息对应的本地事务的状态

(3) 根据本地事务状态，重新Commit或者Rollback

其中，补偿阶段用于解决消息Commit或者Rollback发生超时或者失败的情况。



##### 1.4 延迟队列

> 定时消息（延迟队列）是指消息发送到broker后，不会立即被消费，等待特定时间投递给真正的topic。 broker有配置项messageDelayLevel，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。可以配置自定义messageDelayLevel。注意，messageDelayLevel是broker的属性，不属于某个topic。发消息时，设置delayLevel等级即可：msg.setDelayLevel(level)。level有以下三种情况：

- level == 0，消息为非延迟消息
- 1<=level<=maxLevel，消息延迟特定时间，例如level==1，延迟1s
- level > maxLevel，则level== maxLevel，例如level==20，延迟2h

定时消息会暂存在名为SCHEDULE_TOPIC_XXXX的topic中，并根据delayTimeLevel存入特定的queue，queueId = delayTimeLevel – 1，即一个queue只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费SCHEDULE_TOPIC_XXXX，将消息写入真实的topic。

需要注意的是，定时消息会在第一次写入和调度写入真实topic时都会计数，因此发送数量、tps都会变高。



##### 1.5 顺序消息

> 消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。RocketMQ可以严格的保证消息有序。

> 顺序消息分为全局顺序消息与分区顺序消息，全局顺序是指某个Topic下的所有消息都要保证顺序；部分顺序消息只要保证每一组消息被顺序消费即可。

- 全局顺序 对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。 适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景
- 分区顺序 对于指定的一个 Topic，所有消息根据 sharding key 进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。 Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。 适用场景：性能要求高，以 sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景。



---



#### 2 serverbroker接受消息后

##### 2.1 数据的存储

- 消息集中式存储在commitlog，一个broker只存在一个存储文件
- 异步创建commitqueue文件，包含msgsize，offset，tagcode

##### 2.2 访问顺序

- 先从commitqueue文件查找偏移量，在到commitlog文中查找对应的消息

##### 2.3 消息的清理

- 文件超过72小时成为过期文件
  - 定时任务清除过期文件
  - 超过磁盘的75%删除
  - 超过85%批量清除不管是否过期
  - 超过90%拒绝消息写入

##### 2.4 消息的高可用

##### 2.4.1 主从同步

主从数据同步有两种方式同步复制、异步复制

| 复制方式 | 优点                                                         | 缺点                                                         | 适应场景                 |
| :------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------- |
| 同步复制 | slave保证了与master一致的数据副本，如果master宕机，数据依然在slave中找到其数据和master的数据一致 | 由于需要slave确认效率上会有一定的损失                        | 数据可靠性要求很高的场景 |
| 异步复制 | 无需等待slave确认消息是否存储成功效率上要高于同步复制        | 如果master宕机，由于数据同步有延迟导致slave和master存在一定程度的数据不一致问题 | 数据可靠性要求一般的场景 |

流程

- 从服务器主动建立tcp连接主服务器，每隔5s向主服务器发送commit log文件最大偏移量，拉取还未同步的消息
- 主服务器开启监听端口，监听从服务器发送的消息，主服务器收到从服务器发过来的偏移量进行解析，并返回查找出未同步的消息给从服务器
- 客户端收到主服务器的消息后将这批消息写入commitlog文件中，更新commitlog拉取偏移量，接着继续向主服务器拉取未同步的消息

##### 2.4.2 刷盘

- 刷盘的几种方式

  - 同步磁盘刷新

    > 只有在消息真正持久化至磁盘后RocketMQ的Broker端才会真正返回给Producer端一个成功的ACK响应。同步刷盘对MQ消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多。

  - 异步磁盘刷新

    > 能够充分利用OS的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量。



##### 2.4.3 故障转移





----



#### 3 消费者消费消息时

##### 1 怎么获取topic

> 通过comitqueue偏移量找到commitlog的msg,从messagequeue封装的对象中获取

##### 2 获取失败怎么办

- 重试机制
  
  - 消息的重复消费？
  
  - 重试最大次数失败 - 死信队列
  
    - 死信队列用于处理无法被正常消费的消息。当一条消息初次消费失败，消息队列会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。
  
      RocketMQ将这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。在RocketMQ中，可以通过使用console控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费。
  
  

##### 3 topic或者消费者数量变化如何处理

###### 3.1 rebalance重分配

- 范围分配
- 轮询分配
- 自定义



##### 4 消费过程幂等

RocketMQ无法避免消息重复（Exactly-Once），所以如果业务对消费重复非常敏感，务必要在业务层面进行去重处理。可以借助关系数据库进行去重。首先需要确定消息的唯一键，可以是msgId，也可以是消息内容中的唯一标识字段，例如订单Id等。在消费之前判断唯一键是否在关系数据库中存在。如果不存在则插入，并消费，否则跳过。（实际过程要考虑原子性问题，判断是否存在可以尝试插入，如果报主键冲突，则插入失败，直接跳过）

msgId一定是全局唯一标识符，但是实际使用中，可能会存在相同的消息有两个不同msgId的情况（消费者主动重发、因客户端重投机制导致的重复等），这种情况就需要使业务字段进行重复消费。

##### 5 消费速度慢的处理方式

##### 5.1 提高消费并行度

绝大部分消息消费行为都属于 IO 密集型，即可能是操作数据库，或者调用 RPC，这类消费行为的消费速度在于后端数据库或者外系统的吞吐量，通过增加消费并行度，可以提高总的消费吞吐量，但是并行度增加到一定程度，反而会下降。所以，应用必须要设置合理的并行度。 如下有几种修改消费并行度的方法：

- 同一个 ConsumerGroup 下，通过增加 Consumer 实例数量来提高并行度（需要注意的是超过订阅队列数的 Consumer 实例无效）。可以通过加机器，或者在已有机器启动多个进程的方式。
- 提高单个 Consumer 的消费并行线程，通过修改参数 consumeThreadMin、consumeThreadMax实现。

##### 5.2 批量方式消费

某些业务流程如果支持批量方式消费，则可以很大程度上提高消费吞吐量，例如订单扣款类应用，一次处理一个订单耗时 1 s，一次处理 10 个订单可能也只耗时 2 s，这样即可大幅度提高消费的吞吐量，通过设置 consumer的 consumeMessageBatchMaxSize 返个参数，默认是 1，即一次只消费一条消息，例如设置为 N，那么每次消费的消息数小于等于 N。

##### 5.3 跳过非重要消息

发生消息堆积时，如果消费速度一直追不上发送速度，如果业务对数据要求不高的话，可以选择丢弃不重要的消息。例如，当某个队列的消息数堆积到100000条以上，则尝试丢弃部分或全部消息，这样就可以快速追上发送消息的速度。示例代码如下：

```
    public ConsumeConcurrentlyStatus consumeMessage(
            List<MessageExt> msgs,
            ConsumeConcurrentlyContext context) {
        long offset = msgs.get(0).getQueueOffset();
        String maxOffset =
                msgs.get(0).getProperty(Message.PROPERTY_MAX_OFFSET);
        long diff = Long.parseLong(maxOffset) - offset;
        if (diff > 100000) {
            // TODO 消息堆积情况的特殊处理
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
        // TODO 正常消费过程
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }    
```

##### 5.4 优化每条消息消费过程

举例如下，某条消息的消费过程如下：

- 根据消息从 DB 查询【数据 1】
- 根据消息从 DB 查询【数据 2】
- 复杂的业务计算
- 向 DB 插入【数据 3】
- 向 DB 插入【数据 4】

这条消息的消费过程中有4次与 DB的 交互，如果按照每次 5ms 计算，那么总共耗时 20ms，假设业务计算耗时 5ms，那么总过耗时 25ms，所以如果能把 4 次 DB 交互优化为 2 次，那么总耗时就可以优化到 15ms，即总体性能提高了 40%。所以应用如果对时延敏感的话，可以把DB部署在SSD硬盘，相比于SCSI磁盘，前者的RT会小很多。



##### 6 至少一次

至少一次(At least Once)指每个消息必须投递一次。Consumer先Pull消息到本地，消费完成后，才向服务器返回ack，如果没有消费一定不会ack消息，所以RocketMQ可以很好的支持此特性。

---









