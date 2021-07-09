### mysql 数据库类型

RDBMS即关系数据库管理系统(Relational Database Management System)的特点：

- 1.数据以表格的形式出现
- 2.每行为各种记录名称
- 3.每列为记录名称所对应的数据域
- 4.许多的行和列组成一张表单
- 5.若干的表单组成database  

### mysql 通信协议



**支持tcp/ip协议**

**支持unix socket 不需要网络**

机器 - 端口 - 客户端

远程 - 端口 - 客户端



### mysql 通信方式

1. 单工

   > 数据单向传递

2. 半双工

   > 数据双向传递，同一时间，只允许一个服务传送

3. 全双工

   > 数据双向传递，可以同时传输
   
4. Mysql

   使用的半双工

   默认连接数151，最大2^14 16384



### mysql 运行过程



**<span style=color:red>三层工作流程</span>**



1. 连接层

2. 服务层

3. 存储引擎层



​	<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/MYSQL.png" alt="MYSQL" style="zoom: 67%;" />



**dql  select的工作流程**



**dml  update/delete/insert**



![img](https://raw.githubusercontent.com/YangLiang266/images/master/img/5148507-1a29473c24f0c5b7.png)



### mysql日志系统

> mysql不是每次数据更改都立刻写到磁盘，而是会先将修改后的结果暂存在内存中,当一段时间后，再一次性将多个修改写到磁盘上，减少磁盘io成本，同时提高操作速度。

- mysql通过WAL(write-ahead logging)技术保证事务

  在同一个事务中，每当数据库进行修改数据操作时，将修改结果更新到内存后，会在redo log添加一行记录记录“需要在哪个数据页上做什么修改”，并将该记录状态置为prepare，等到commit提交事务后，会将此次事务中在redo log添加的记录的状态都置为commit状态，之后将修改落盘时，会将redo log中状态为commit的记录的修改都写入磁盘。

  

- redo log记录方式

  redolog的大小是固定的，在mysql中可以通过修改配置参数innodb_log_files_in_group和innodb_log_file_size配置日志文件数量和每个日志文件大小，redolog采用循环写的方式记录，当写到结尾时，会回到开头循环写日志。如下图

  <img src="https:////upload-images.jianshu.io/upload_images/5148507-bef2727de72b311f.png?imageMogr2/auto-orient/strip|imageView2/2/w/854/format/webp" alt="img" style="zoom:50%;" />

  

  write pos表示日志当前记录的位置，当ib_logfile_4写满后，会从ib_logfile_1从头开始记录；check point表示将日志记录的修改写进磁盘，完成数据落盘，数据落盘后checkpoint会将日志上的相关记录擦除掉，即write pos->checkpoint之间的部分是redo log空着的部分，用于记录新的记录，checkpoint->write pos之间是redo log待落盘的数据修改记录。当writepos追上checkpoint时，得先停下记录，先推动checkpoint向前移动，空出位置记录新的日志。

  有了redo log，当数据库发生宕机重启后，可通过redo log将未落盘的数据恢复，即保证已经提交的事务记录不会丢失。

  有了redo log，为啥还需要binlog呢？

> 1、redo log的大小是固定的，日志上的记录修改落盘后，日志会被覆盖掉，无法用于数据回滚/数据恢复等操作。
>  2、redo log是innodb引擎层实现的，并不是所有引擎都有。

- **基于以上，binlog必不可少**

> 1、binlog是server层实现的，意味着所有引擎都可以使用binlog日志
>  2、binlog通过追加的方式写入的，可通过配置参数max_binlog_size设置每个binlog文件的大小，当文件大小大于给定值后，日志会发生滚动，之后的日志记录到新的文件上。
>  3、binlog有两种记录模式，statement格式的话是记sql语句， row格式会记录行的内容，记两条，更新前和更新后都有。

binlog和redo log必须保持一致，不允许出现binlog有记录但redolog没有的情况，反之亦然。之前说过在一个事务中，redolog有prepare和commit两种状态，所以，在redolog状态为prepare时记录binlog可保证两日志的记录一致



#### 两者区别

1. redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用。
2. redo log是物理日志，记录的是"在某个数据页上做了什么修改"；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如"给ID=2这一行的c字段加1 "。
3. redo log是循环写的，空间固定会用完；binlog是可以追加写入的。"追加写"是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志



redolog 物理层数据日志

binlog 服务层被所有存储引擎使用，sql语句记录，逻辑日志



<img src="https://raw.githubusercontent.com/YangLiang266/images/master/img/mysql优化.png" alt="mysqlOS" style="zoom:80%;" />