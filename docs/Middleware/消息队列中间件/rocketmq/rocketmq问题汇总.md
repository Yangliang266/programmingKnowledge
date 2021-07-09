rocketmq 是否是分片，如何实现数据的扩展

rocketmq 消费端的单线程设置

消费端去重（全局表实现）一条消息无论重试多少次，这些重试消息的 Message ID，key 不会改变。
   消费重试只针对集群消费方式生效；



RocketMQ不保证消息不重复，如果你的业务需要保证严格的不重复消息，需要你自己在业务端去重

```
// 并发消息消费逻辑实现类
org.apache.rocketmq.client.impl.consumer.ConsumeMessageConcurrentlyService;
// 顺序消息消费逻辑实现类
org.apache.rocketmq.client.impl.consumer.ConsumeMessageOrderlyService;
```



并发就是多线程吗

wget https://archive.apache.org/dist/rocketmq/4.7.1/rocketmq-all-4.7.1-bin-release.zip

```
mkdir -p /usr/local/midmn/rocketmq/store/broker-a /usr/local/midmn/rocketmq/store/broker-a/consumequeue /usr/local/midmn/rocketmq/store/broker-a/commitlog /usr/local/midmn/rocketmq/store/broker-a/index /usr/local/midmn/rocketmq/broker-a/logs 
```

mkdir -p /usr/local/midmn/rocketmq/store/broker-a /usr/local/midmn/rocketmq/store/broker-a/consumequeue /usr/local/midmn/rocketmq/store/broker-a/commitlog /usr/local/midmn/rocketmq/store/broker-a/index /usr/local/midmn/rocketmq/broker-a/logs 



```
#Broker 对外服务的监听端口
listenPort=10911
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#nameServer地址，分号分割
namesrvAddr=172.16.111.59:9876
#存储路径
storePathRootDir=/usr/local/midmn/rocketmq/store/broker-a
#commitLog 存储路径
storePathCommitLog=/usr/local/midmn/rocketmq/store/broker-a/commitlog
#消费队列存储路径
storePathConsumeQueue=/usr/local/midmn/rocketmq/store/broker-a/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/midmn/rocketmq/store/broker-a/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/midmn/rocketmq/store/broker-a/checkpoint
#abort 文件存储路径
abortFile=/usr/local/midmn/rocketmq/store/broker-a/abort
```

```
nohup sh mqbroker -c /usr/local/midmn/rocketmq/conf/broker.conf &
```

