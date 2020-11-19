## 高并发引起的一些问题

### 分布式锁

#### 1 解决问题

1. 解决跨进程访问的共享数据影响数据一致性
2. 微服务中集群的搭建跨进程访问

#### 2 可用技术

1. zookeeper 节点的有序性和唯一性
2. redis setnx特性

#### 3 redis客户端解决方案

1. jedis 
   1. 获得锁
      1. uuid -> value
      2. lockName - > key
      3. expire
      4. jedis创建连接
      5. 实现setnx
      6. 关闭连接
   2. 释放锁
      1. watch监听
      2. transport
      3. exec
      4. dele
      5. unwatch
      6. 关闭连接
   3. lua实现
      1. 特性实现原子性，不被打断操作
      2. 编写脚本
      3. eva调用，传入lockname，expire
2. redisson
   1. 获得锁
      1. trylock
   2. 释放锁
      1. unlock





### redis数据一致性

#### 1 缓存使用场景

1. 操作redis后数据库
   1. 定时任务，删除redis，读取数据库
   2. 过期时间，删除redis，读取数据库
   3. 异常（数据库更新失败）
      1. 并发导致redis获取旧值，当数据库操作成功，则redis还是旧值
      2. 延迟双删
2. 操作数据库后redis
   1. 正常
   2. 异常（删除redis失败）
      1. 二次删除尝试- 对业务代码入侵
         1. 消息队列传输？
      2. 服务监听数据库binlog，cannal