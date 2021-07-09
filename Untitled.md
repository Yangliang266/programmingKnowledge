curl -L -O https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-7.10.1-linux-x86_64.tar.gz





1 nosql

支持高并发



2 压缩方式

List

quicklist快速列表

ziplist和linkedlist的结合体

ziplist存有相互指向的列表，每个列表又有一个指针指向数据的存储

hash

ziplist

连续内存块组成的压缩列表

hashtable

数组+链表



3 虚拟槽



4 一致性hash算法

取2的32次方的模形成一个闭合圆环，每个节点分布在圆环上

每个服务计算多个hash，分布在圆环，使得在节点少的情况下，数据能够分布均匀

5 分布式锁- lua

锁的创建与释放需要一个合理的判断流程且不能中断，lua脚本保持原子性即执行过程中，其他命令处于阻塞状态





git.gupaoedu.com/vip/



多服务同一域名，静态缓存，灰度发布

host文件配置的原因

- 正常走域名解析
- 域名映射成服务器地址





sh /usr/local/rocketmq/rocketmq-all-4.7.1-bin-release/bin/mqshutdown namesrv

nohup sh mqnamesrv /dev/null 2>&1 &

nohup sh mqbroker &



nohup sh mqbroker -c /usr/local/rocketmq/rocketmq-all-4.7.1-bin-release/conf/broker.conf >/usr/local/rocketmq/rocketmq-all-4.7.1-bin-release/temp/broker.log 2>&1 &

nohup sh mqbroker -n 192.168.89.138:9876 /dev/null 2>&1 &



namesrvAddr= 139.196.44.115:9876





#namesrvAddr= 172.16.111.59:9876



![image-20210124162029400](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20210124162029400.png)





#### 先说感悟

> 七大软件设计原则在实际生产环境下不可能独存，必有相结合之处，类似周易八卦，一生二，两仪生四象，所谓一阴一阳之谓道，各自的组合会产生不同的效果，适用于各自的生产环境，不可强求，只需最后固本守元，不偏离航道，自己把握即可。



#### 下面阐述自己理解的七大原则本质

1. 开闭原则

   > 只出不入

2. 依赖倒置原则

   > 五行的运行依赖于天地抽象，以天地为基础

3. 单一职责原则和接口隔离原则

   > 可隐喻为: 前者为五行有各自的实现,后者为气，气是调动五行循环往复的根本即引擎。

4. 迪米特原则

   > 五行各有各的特征状态

5. 里氏替换原则

   > 不影响本源，隐喻为：一棵树，树根为本源，树干，树枝，树叶都是其衍生。
   >
   > 因为促进其生长的是水分，所以水分不管是雨水还是河水都不影响其生长。
   >
   > 但是吸收的水分，只可能生出树干，树枝，树叶而不可能长出花，草。
   >
   > 即宽进严出

6. 合成复用原则

   > 一阴一阳相互包容，进行衍生



#### 现象级别

七大设计原则，可以看作是七大现象，现象没有对错之分，和谐共处独有的场景会有它的容身之地，不管是七大设计原则，还是代码的设计，在我看来，都应把握核心，在此基础衍生扩展，不偏离本源





​																																						---- over alan 2021/03/14 爱你一生一世 柳柳



![image-20210314210432623](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/image-20210314210432623.png)





![image-20210314221920223](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/image-20210314221920223.png)



![image-20210314221937088](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/image-20210314221937088.png)