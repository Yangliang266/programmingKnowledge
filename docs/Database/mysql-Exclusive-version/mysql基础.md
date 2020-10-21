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



​	连接 - 缓存 - 语句解析，词法解析 - 预处理器 - 查询优化器 - 执行计划 - 执行引擎 - 存储引擎



**dql  select的工作流程**

<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/mysql-dql流程.png" alt="img" style="zoom:80%;" />

**dml  update/delete/insert**

<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/mysql-dml流程.PNG" alt="img" style="zoom:80%;" />



redolog 物理层数据日志

binlog 服务层被所有存储引擎使用，sql语句记录，逻辑日志



<img src="https://raw.githubusercontent.com/YangLiang-SoftWise/images/master/img/mysql优化.png" alt="mysqlOS" style="zoom:80%;" />