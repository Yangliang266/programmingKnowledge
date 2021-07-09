### zk，elasticsearch进程频繁被杀



#### 猜想一

服务器内存不够导致进程结束

#### 实际

加服务器内存

#### 后续

进程依然中止



#### 猜想二

elastic的jvm设置过小，导致oom，强制进程结束

#### 实际

更改jvm大小

#### 后续

进程依然中止



#### 猜想三

为什么zookeeper进程也会强制结束，是否未采取守护模式启动

#### 实际

systemctl 守护启动zk，且设置开机自启

##### 流程

- /etc/systemd/system文件夹下创建zookeeper.service

  ```java
  [Unit]
  Description=zookeeper.service
  After=network.target
  
  [Service]
  Type=forking
  ExecStart=/usr/local/micromn/zookeeper-3.4.14/bin/zkServer.sh start
  ExecStop=/usr/local/micromn/zookeeper-3.4.14/bin/zkServer.sh stop
  ExecReload=/usr/local/micromn/zookeeper-3.4.14/bin/zkServer.sh restart
  
  [Install]
  WantedBy=multi-user.target
  ```


- 设置文件权限chmod 755 zookeeper.service
- 在zkEnv.sh 加上`JAVA_HOME=/usr/local/jdk/jdk1.8.0_11`
- systemctl daemon-reload *#重载服务配置项*
- systemctl enable zookeeper	*#所有服务通用*
- systemctl start zookeeper	*#启动zookeeper*

#### 后续

进程依然中止



#### 猜想四

服务器重启其他进程未关闭，导致pid重复，引起进程终止

#### 流程

更改/tmp相关进程文件的pid

#### 后续

进程依然异常终止



#### 猜想五

服务器cpu高达100%，且报警有挖矿程序，有恶意访问地址

每次命令都会出现You have new mail in /var/spool/mail/root

#### 流程

- top 未察觉到异常进程占用cpu

- 执行`crontab -l` 查看，发现多了一条定时任务 

  > */30 * * * * sh /etc/newdat.sh >/dev/null 2>&1

- 安装阿里云agent - 无效（垃圾）

- /etc 文件夹有`newinit.sh,newbat.sh`等一系列文件同一时间创建，同时修改root

- **百度为挖矿程序GuardMiner病毒**（cpu占用罪魁祸首）

- 删除newinit.sh文件

  - rm: cannot remove `xxx’: Operation not permitted
  
    > lsattr xxx  查看文件属性(xxx为文件名)
    >
    > chattr -a xxx 或者 chattr -i xxx
    >
    > rm -rf xxx

#### 后续

暂时未复发，后续观察

![image-20210115203400810](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20210115203400810.png)

#### 



#### 为什么会中招

- 为了方便在任何网络情况下都能访问服务器所有的访问ip都设置为可访问

- 外网利用redis等一系列无限制端口访问入侵，redis的权限又是root，进而进行控制

  





nohub ./packebeat >/usr/local/datamn/packetbeat-7.10.1-linux-x86_64/log.txt 2>&1 &