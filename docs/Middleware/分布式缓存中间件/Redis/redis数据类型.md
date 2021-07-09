## 数据的存储形式kv

<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201214164623436.png" alt="image-20201214164623436" style="zoom:50%;" />

```
dictEntry
	key -> sds(key关键字)
	val -> redisObject -> ptr -> sds（string）
							  -> 其他数据类型底层编码
```



### 1 String类型

**存储类型**

> 支持int/float/string三种数据类型

**操作命令**

```xml
# 获取指定范围的字符
get range yang 0 1
# 获取值长度
strlen yang
# 字符串追加
append yang good
# 设置多个值
mset yang 2 yang1 3
# 获取多个值
mget yang yang1
# 设置值key存在，不成功
setnx yang 1
# 多参数设置，在释放锁失败，过期失败，保证原子性
set yang2 WW EX 5 NX

```



**存储原理**

> SDS作为String类型的内部编码简称为简单动态字符串

1. SDS存在的意义

   > 预分配：提前分配好相应大小的字符串内存
   >
   > 二进制安全：C 语言中的字符串是以 ”\0“ 作为字符串的结束标记。而 SDS 是使用 len 的长度来标记字符串的结束
   >
   > 惰性释放：如果缩短 SDS 的字符串长度，redis并不是马上减少 SDS 所占内存。只是增加 free 的长度，分配给下一个需要内存的程序。同时向外提供 API 。真正需要释放的时候，才去重新缩小 SDS 所占的内存

2. SDS形态的转换

   > int/float/string不管哪种类型进行添加append，都会从相应的编码转变为raw编码
   >
   > int/embstr/raw之间的转换：
   >
   > int不再是整数 -> raw
   >
   > int大小超过2^63-1 -> embstr
   >
   > embstr超过44个字节 -> raw

   

**应用**

- **缓存**
  String 类型
  例如：热点数据缓存（例如报表，明星出轨），对象缓存，全页缓存。
  可以提升热点数据的访问速度。  

- **数据共享分布式**
  STRING 类型，因为 Redis 是分布式的独立服务，可以在多个应用之间共享
  例如：分布式 Session

```XML
<dependency>
<groupId>org.springframework.session</groupId>
<artifactId>spring-session-data-redis</artifactId>
</dependency>
```

- **分布式锁**
  STRING 类型 setnx 方法，只有不存在时才能添加成功，返回 true  

```JAVA
public Boolean getLock(Object lockObject){
jedisUtil = getJedisConnetion();
boolean flag = jedisUtil.setNX(lockObj, 1);
if(flag){
expire(locakObj,10);
}
return flag;
}
public void releaseLock(Object lockObject){
del(lockObj);
}
```

- **全局 ID**
  INT 类型，INCRBY，利用原子性
  incrby userid 1000
  （分库分表的场景，一次性拿一段）  

- **计数器**
  INT 类型，INCR 方法
  例如：文章的阅读量，微博点赞数，允许一定的延迟，先写入 Redis 再定时同步到
  数据库。

- **限流**
  INT 类型，INCR 方法
  以访问者的 IP 和其他信息作为 key，访问一次增加一次计数，超过次数则返回 false。  

- **位统计**
  String 类型的 BITCOUNT（1.6.6 的 bitmap 数据结构介绍）。
  字符是以 8 位二进制存储的。

  ```
  set k1 a
  setbit k1 6 1
  setbit k1 7 0
  get k1
  a 对应的 ASCII 码是 97，转换为二进制数据是 01100001
  b 对应的 ASCII 码是 98，转换为二进制数据是 01100010
  因为 bit 非常节省空间（1 MB=8388608 bit），可以用来做大数据量的统计。
  例如：在线用户统计，留存用户统计
  
  setbit onlineusers 0 1
  setbit onlineusers 1 1
  setbit onlineusers 2 0
  支持按位与、按位或等等操作。  
  ```

  






### 2 Hash

**1 存储类型**

> Redis 的 Hash 本身也是一个 KV 的结构，类似于 Java 中的 HashMap  
>
> 外层的哈希（Redis KV 的实现）只用到了 hashtable。当存储 hash 数据类型时，我们把它叫做内层的哈希。内层的哈希底层可以使用两种数据结构实现 

**操作命令**

```xml
hset h1 f 6
hset h1 e 5
hmset h1 a 1 b 2 c 3 d 4
hget h1 a
hmget h1 a b c d
hkeys h1
hvals h1
hgetall h1
# key操作
hget exists h1
hdel h1
hlen h1
```

redis.config

| 参数                         | 含义                                  |
| ---------------------------- | ------------------------------------- |
| hash-max-ziplist-value 64    | ziplist 中最大能存放的值长度          |
| hash-max-ziplist-entries 512 | ziplist 中最多能存放的 entry 节点数量 |

**2 存储原理**

1. ziplist(压缩列表)

   > 由连续内存快组成的双向链表，存储上个节点的长度和当前节点的长度，通过牺牲部分读写性能，来换取高效的内存空间利用率，是一种时间换空间的思想。只用在字段个数少，字段值小的场景里面  

   ![image-20201215095003370](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215095003370.png)

   **何时适用**

   > 一个hash对象保存的field数量<512个
   >
   > 一个hash对象中所有field和value字符串长度<64byte

   一个哈希对象超过配置的阈值（键和值的长度有>64byte，键值对个数>512 个）时，会转换成哈希表（hashtable）  

2. hashtable

   > 在 Redis 中，hashtable 被称为字典（dictionary），它是一个数组+链表的结构  

   ![image-20201215100921066](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215100921066.png)

   **为什么要定义两个哈希表呢？ht[2]**
   redis 的 hash 默认使用的是 ht[0]，ht[1]不会初始化和分配空间。
   哈希表 dictht 是用链地址法来解决碰撞问题的。在这种情况下，哈希表的性能取决于它的大小（size 属性）和它所保存的节点的数量（used 属性）之间的比率：

   - 比率在 1:1 时（一个哈希表 ht 只存储一个节点 entry），哈希表的性能最好；
   - 如果节点数量比哈希表的大小要大很多的话（这个比例用 ratio 表示，5 表示平均一个 ht 存储 5 个 entry），那么哈希表就会退化成多个链表，哈希表本身的性能优势就不再存在。
   - 在这种情况下需要扩容。Redis 里面的这种操作叫做 rehash。
     rehash 的步骤：
     1. 为字符 ht[1]哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及 ht[0]当前包含的键值对的数量。
        扩展：ht[1]的大小为第一个大于等于 ht[0].used*2。
     2. 将所有的 ht[0]上的节点 rehash 到 ht[1]上，重新计算 hash 值和索引，然后放入指定的位置。
     3. 当 ht[0]全部迁移到了 ht[1]之后，释放 ht[0]的空间，将 ht[1]设置为 ht[0]表，并创建新的 ht[1]，为下次 rehash 做准备。  



**应用**

- String
  String 可以做的事情，Hash 都可以做。

- 存储对象类型的数据
  比如对象或者一张表的数据，比 String 节省了更多 key 的空间，也更加便于集中管理。  

- 购物车  

  key：用户 id；field：商品 id；value：商品数量。
  +1：hincr。-1：hincr。删除：hdel。全选：hgetall。商品数：hlen。  





### 3 List

**1 存储类型**

> 存储有序的字符串，从左到右，元素可以重复，可以充当队列和栈的角色  。

**2 操作命令**

```xml
元素增减：
lpush queue a
lpush queue b c

rpush queue d e
lpop queue
rpop queue
blpop queue
brpop queue
取值
lindex queue 0
lrange queue 0 -1
```

<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215101849023.png" alt="image-20201215101849023" style="zoom:50%;" />

redis.config

| 参数                            | 含义                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| list-max-ziplist-size（fill）   | 正数表示单个 ziplist 最多所包含的 entry 个数。 负数代表单个 ziplist 的大小， 默认 8k。 -1： 4KB； -2： 8KB； -3： 16KB； -4： 32KB； -5： 64KB |
| list-compress-depth（compress） | 压缩深度， 默认是 0。 1： 首尾的 ziplist 不压缩； 2： 首尾第一第二个 ziplist 不压缩， 以此类推 |



**3 存储原理**

> quicklist（快速列表）是 ziplist 和 linkedlist 的结合体。  

![image-20201215104809953](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215104809953.png)

> quicklistNode 中的*zl 指向一个 ziplist，一个 ziplist 可以存放多个元素  



**4 应用**

- 用户消息时间线 timeline
  因为 List 是有序的，可以用来做用户时间线  

- 消息队列
  List 提供了两个阻塞的弹出操作：BLPOP/BRPOP，可以设置超时时间。
  BLPOP：BLPOP key1 timeout 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
  BRPOP：BRPOP key1 timeout 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
  队列：先进先出：rpush blpop，左头右尾，右边进入队列，左边出队列。
  栈：先进后出：rpush brpop  



### 4 set

**1 存储类型**

> String 类型的无序集合，最大存储数量 2^32-1  

**2 操作命令**

```xml
添加一个或者多个元素
sadd myset a b c d e f g
获取所有元素
smembers myset
统计元素个数
scard myset
随机获取一个元素
srandmember key
随机弹出一个元素
spop myset
移除一个或者多个元素
srem myset d e f
查看元素是否存在
sismember myset a
```

**存储原理**

> Redis 用 intset 或 hashtable 存储 set。如果元素都是整数类型，就用 inset 存储。
> 如果不是整数类型，就用 hashtable（数组+链表的存来储结构）。
> 问题：KV 怎么存储 set 的元素？key 就是元素的值，value 为 null。
> 如果元素个数超过 512 个，也会用 hashtable 存储。  

```xml
redis.conf
set-max-intset-entries 512
```

**应用**

- 抽奖
  随机获取元素
  spop myset  

- 点赞、 签到、 打卡  

<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215110609769.png" alt="image-20201215110609769" style="zoom: 67%;" />

```
这条微博的 ID 是 t1001，用户 ID 是 u3001。
用 like:t1001 来维护 t1001 这条微博的所有点赞用户。
点赞了这条微博：sadd like:t1001 u3001
取消点赞：srem like:t1001 u3001
是否点赞：sismember like:t1001 u3001
点赞的所有用户：smembers like:t1001
点赞数：scard like:t1001
比关系型数据库简单许多  
```



### 5 zset

**存储类型**

> sorted set，有序的 set，每个元素有个 score。
> score 相同时，按照 key 的 ASCII 码排序。

数据结构对比：

| 数据结构      | 是否允许重复元素 | 是否有序 | 有序实现方式 |
| ------------- | ---------------- | -------- | ------------ |
| 列表 list     | 是               | 是       | 索引下标     |
| 集合 set      | 否               | 否       | 无           |
| 有序集合 zset | 否               | 是       | 分值 score   |

**操作命令**

```xml
添加元素
zadd myzset 10 java 20 php 30 ruby 40 cpp 50 python
获取全部元素
zrange myzset 0 -1 withscores
zrevrange myzset 0 -1 withscores
根据分值区间获取元素
zrangebyscore myzset 20 30
移除元素
也可以根据 score rank 删除
zrem myzset php cpp
统计元素个数
zcard myzset
分值递增
zincrby myzset 5 python
根据分值统计个数
zcount myzset 20 60
获取元素 rank
zrank myzset java
获取元素 score
zscore myzset java


也有倒序的 rev 操作（reverse）
```

**存储原理**

- 同时满足以下条件时使用 ziplist 编码：元素数量小于 128 个,所有 member 的长度都小于 64 字节  

```
对应 redis.conf 参数：
zset-max-ziplist-entries 128
zset-max-ziplist-value 64  
```



- 在 ziplist 的内部，按照 score 排序递增来存储。插入的时候要移动之后的数据。

- 超过阈值之后，使用 skiplist+dict 存储  

  **什么是 skiplist？  **

  路径是沿着下图中标红的指针所指向的方向进行的：

![image-20201215112705806](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201215112705806.png)

1. 23 首先和 7 比较，再和 19 比较，比它们都大，继续向后比较。
2. 但 23 和 26 比较的时候，比 26 要小，因此回到下面的链表（原链表），与 22比较。
3. 23 比 22 要大，沿下面的指针继续向后和 26 比较。23 比 26 小，说明待查数据 23 在原链表中不存在在这个查找过程中，由于新增加的指针，我们不再需要与链表中每个节点逐个进行比较了。需要比较的节点数大概只有原来的一半。这就是跳跃表。



- 为什么不用 AVL 树或者红黑树？因为 skiplist 更加简洁。  



**应用**

- 排行榜  

  id 为 6001 的新闻点击数加 1：zincrby hotNews:20190926 1 n6001
  获取今天点击最多的 15 条：zrevrange hotNews:20190926 0 15 withscores  