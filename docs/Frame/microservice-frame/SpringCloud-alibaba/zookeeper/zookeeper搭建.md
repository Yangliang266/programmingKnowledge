**一 下载 

![zookeeper-3.4.14.tar.gz](https://raw.githubusercontent.com/YangLiang266/images/master/img/attachment.png)

复制zoo.cfg文件且修改

`dataDir=/tmp/zookeeper/dat`

`dataLogDir=/tmp/zookeeper/log`



**二 netstat -ntlp 查看服务端口**

**1 bin/zkServer.sh status 查看服务状态或者（systemctl status firewalld）**

1. 2181端口操作(一般被默认监听不需要改动netstat -ntlp查询)

   ```
   查看2181端口
   
   firewall-cmd --query-port=2181/tcp
   
   开启2181端口
   
   firewall-cmd --permanent --add-port=8081/tcp
   
   移除端口
   
   firewall-cmd --permanent --remove-port=8081/tcp
   
   重启防火墙(修改配置后要重启防火墙)
   
   firewall-cmd --reload
   ```

   

2. 防火墙的两种方法操作

   ```
   开启:service iptables start
   
   关闭:service iptables stop
   
   状态:service iptables status
   
   开启:systemctl start firewalld
   
   关闭:systemctl status firewalld
   
   状态:systemctl stop firewalld
   
   firewall-cmd -server start
   ```

   

**三 启动zk服务**

1 bin/zkServer.sh start 启动服务

2 bin/zkCli.sh -server 192.168.89.134:2181 启动服务端口

3 打开可视化工具



**四 zk操作**

查看：ls / 创建赋值：create /cat hadoop		#/bhz为节点，hadoop 是存放的数据（值） 

获取数据：get /cat 设值：set /cat 

递归删除节点：rmr/path 

删除某个指定节点：delete /path/child