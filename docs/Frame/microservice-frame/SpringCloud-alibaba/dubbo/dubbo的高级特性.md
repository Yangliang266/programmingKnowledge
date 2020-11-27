## 多协议支持

- dubbo
- rest
- thirft

##### 对内

- dubbo
- thirft

##### 对外

- rest/http

#### rest

spring mvc

jax-rs

- dubbo -> RESTeazy



## 负载均衡(基于客户端的负载均衡)

#### 随机random

加权随机

1 根据(服务性能)权重大小生成区间

2 随机生成一个随机数

3 落在哪个区间就去分配



#### roundrobin (轮询)

加权轮询

1 根据(服务性能)权重大小，请求落在相应的服务上



#### LeastActiveLoadBalance 最小活跃数

活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求。此时应优先将请求分配给该服务提供者。在具体实现中，每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下降的也越快，此时这样的服务提供者能够优先获取到新的服务请求、这就是最小活跃数负载均衡算法的基本思想。除了最小活跃数，LeastActiveLoadBalance 在实现上还引入了权重值。所以准确的来说，LeastActiveLoadBalance 是基于加权最小活跃数算法实现的。举个例子说明一下，在一个服务提供者集群中，有两个性能优异的服务提供者。某一时刻它们的活跃数相同，此时 Dubbo 会根据它们的权重去分配请求，权重越大，获取到新请求的概率就越大。如果两个服务提供者权重相同，此时随机选择一个即可。



## 服务降级

在 spring 配置文件中按以下方式配置：

```xml
<dubbo:reference interface="com.foo.BarService" mock="true" />
// 此方法在当前接口包中，寻找类+Mock的文件
```

或

```xml
<dubbo:reference interface="com.foo.BarService" mock="com.foo.BarServiceMock" />
```

在工程中提供 Mock 实现 [2](https://dubbo.apache.org/zh/docs/v2.7/user/examples/local-mock/#fn:2)：

```java
package com.foo;
public class BarServiceMock implements BarService {
    public String sayHello(String name) {
        // 你可以伪造容错数据，此方法只在出现RpcException时被执行
        return "容错数据";
    }
```



## 服务容错（基于客户端的容错）

集群调用失败时，Dubbo 提供的容错方案

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

![cluster](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/cluster.jpg)

各节点关系：

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List<Invoker>` ，但与 `List` 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

### Failover Cluster

失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含一次)。

### Failfast Cluster[ ](https://dubbo.apache.org/zh/docs/v2.7/user/examples/fault-tolerent-strategy/#failfast-cluster)

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

### Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

### Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。（保证最终一致性）

### Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设最大并行数。

### Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。



## dubbo泛化



## 常见配置

- 启动时检查check=true
- 主机绑定问题

- 配置的优先级问题

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/1015156-20160927172338172-1904551545.png)

- 上图中

  以timeout为例，显示了配置的查找（优先级）顺序

  ，其它retries, loadbalance, actives等类似。

  - **方法级**优先，**接口**级次之，全局配置再次之。
  - 如果级别一样，则**消费方**优先，提供方次之。

- 其中，服务提供方配置，通过URL经由注册中心传递给消费方。

- 建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置。

 

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/1015156-20160927172733375-1452388768.png)

覆盖策略：

- **JVM启动-D参数优先，这样可以使用户在部署和启动时进行参数重写**，比如在启动时需改变协议的端口。
- XML次之，如果在XML中有配置，则dubbo.properties中的相应配置项无效。
- Properties最后，相当于缺省值，只有XML没有配置时，dubbo.properties的相应配置项才会生效，通常用于共享公共配置，比如应用名





> dubbo 是基于url驱动的，所有的配置都会放在url上

springmvc 来构建rest？

每个对象创建都有hash？

内网ip和外网ip?





