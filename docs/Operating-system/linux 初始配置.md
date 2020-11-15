#### 系统环境准备

首先，centOs上去，确认网卡
vi /etc/sysconfig/network-scripts/ifcfg-eth0
ONBOOT=yes

> ONBOOT是指明在系统启动时是否激活网卡，只有在激活状态的网卡才能去连接网络，进行网络通讯
> 其中eth0是设备名,可能是en33  

重启生效

service network restart
ping www.baidu.com 接收到返回结果则成功。



##### 1.确认系统内存

要是在生产环境下确定你的内存在16G以上 ，因为我们的ES7.X默认需要这么大的内存.但是我们条件有
限，就分个4G吧。

> 参考书籍:[Elasticsearch: 权威指南]节选
> https://www.elastic.co/guide/cn/elasticsearch/guide/current/hardware.html

> ![image-20201114223353763](https://raw.githubusercontent.com/Yangliang266/images/master/img/image-20201114223353763.png)

上文是基于ES2.x版本,但是7.X版本与2.X版本内存要求没有明显变化.

#### 2.确定防火墙

如果服务器涉及到一些端口开放，特别是要自动连接无限制端口，那么我们就得清除Linux的防火墙。
但是绝大多数的Linux服务器，默认的防火墙是有OUTPUT策略。

> 输入命令：iptables -L

> Chain OUTPUT (policy ACCEPT)  

如果有这个策略在，那么就应该先改掉这个。如果不改掉，就清空防火墙，那么你服务器就打不开了，需要重新配置网络才行。

> 把默认策略改成ACCEPT。 iptables -P INPUT ACCEPT 如果不确定就再看看现在的服务器配置，里面是否有这个信息：

> Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
> 如果有上面这句，那么就没问题了，然后再去清除防火墙。
> 输入命令：
> /sbin/iptables -F
> 或iptables -F 删除防火墙规则
> 现在防火墙就关闭清空了，端口任意连接了。

```
\#查询防火墙规则列表
iptables -L;
\#把默认策略改成ACCEPT
iptables -P INPUT ACCEPT;
\#删除防火墙规则
iptables -F;
\#下面有详细描述
iptables -I INPUT -s 192.168.0.0 -j ACCEPT ;  
```

> 删除防火墙规则
> iptables设置句，-A INPUT只在INPUT链中插入一条规则，-s匹配源地址，这里的0/0指可以是任何地址，-i指定网络接口 -d匹配目的地址 -p匹配协议类型，-j指定要采取的操作，这里ACCEPT表示允许连接，
> 这句语句的意思就是配置网络接口eth0允许来自任何地址的目的地址是192.168.0.0的TCP数据包联机。

##### 3.SEliunx

首先,SELinux策略是白名单原则，所以你需要非常清楚你的各项操作都需要哪些访问权限，这个好像数量有点多了,不是专业的运维这东西确实有点麻烦.

- getenforce命令这个是查看当前SELinux的运行模式的指令
  SEliunx有三种模式.分别是
  - Enforcing：强制模式。代表SELinux在运行中，且已经开始限制domain/type之间的验证关系
  - Permissive：宽容模式。代表SELinux在运行中，不过不会限制domain/type之间的验证关系，即使验证不正确，进程仍可以对文进行操作。不过如果验证不正确会发出警告
  - Disabled：关闭模式。SELinux并没有实际运行我们这里需要关闭.
    

```
SEliunx模式快捷转换

\# 转换为Permissive宽容模式
setenforce 0
\# 转换为Enforcing强制模式
setenforce 1  
```



- 注意事项：setenforce无法设置SELinux为Disabled模式
  演示：

  - 如何关闭SEliunx

    > 我们既然要永久关闭selinux，只能通过修改配置文件。
    > 因为selinux开机直接被内核整合，所以selinux没有提供服务接口，也就是说你在/etc/init.d
    > 里是找不到selinux的服务的。
    > selinux的配置文件是/etc/selinux/config
    > vim /etc/selinux/config打开selinux配置文件
    > 打开后按i插入
    > 修改参数部分
    > SELINUX=参数
    > 参数可选（enforcing、permissive、disabled）
    > 输入disabled保存
    > 直接查看运行状态或者重启reboot



```
\#打开selinux配置文件
vim /etc/selinux/config
修改配置文件，SELINUX=参数
参数可选（enforcing、permissive、disabled）
输入disabled保存
重启命令，生产环境禁止使用
reboot  
```

##### 4.时区

> 确认时区：data
> 如果时区是EST或者其他，要修改为CST（中国时区）
> cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

##### 5.检查原有的是否自带openJDK

java -version 判断centOS是否自带JDK
自带JDK则需要删除

> rpm -qa | grep java 这些都是自带的java文件
> rpm -e --nodeps java-1.8.0-openjdk-1.8.0.111-2.6.7.8.el7.x86_64

表示删除你所找到的openJDK，请注意。上面JDK版本与你自带的JDK版本对应
执行 rpm -qa | grep java 无文件，表示删除干净



##### 6.然后安装你需要安装的JDK版本  

解压

```
tar -zxvf java-1.8.0-openjdk-1.8.0.111-2.6.7.8.el7.x86_64
```

配置环境变量JAVA_HOME  

```
vi /etc/profile
在/etc/profile下添加如下内容:
export JAVA_HOME=/jdk/jdk1.8.0_261
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

使环境变量生效  

```
source /etc/profile
```

查看环境变量  

```
echo $PATH
查看我们配置的环境变量是否生效
```

测试JDK是否生效  

```
java -version
```



##### 7.配置操作系统  

注意：这里的话，需要 使用root权限，不然所有的用户都需要单独设置一遍。

进程线程数与文件句柄数
找到limits.conf文件  

```
vi /etc/security/limits.conf
```

进入文件，在文件末尾增加  

![image-20201114225844985](https://raw.githubusercontent.com/Yangliang266/images/master/img/image-20201114225844985.png)



```
* soft nproc 131072
* hard nproc 131072
* soft nofile 131072
* hard nofile 131072
```



> 设置限制数量，第一列表示用户，* 表示所有用户
> soft xxx : 代表警告的设定，可以超过这个设定值，但是超过后会有警告。
> hard xxx : 代表严格的设定，不允许超过这个设定的值。
> nproc : 是操作系统级别对每个用户创建的进程数的限制
> nofile : 是每个进程可以打开的文件数的限制
> soft nproc ：单个用户可用的最大进程数量(超过会警告);
> hard nproc：单个用户可用的最大进程数量(超过会报错);
> soft nofile ：可打开的文件描述符的最大数(超过会警告);
> hard nofile ：可打开的文件描述符的最大数(超过会报错);
> 举例：soft 设为1024，hard设为2048 ，则当你使用数在1~1024之间时可以随便使用，
> 1024~2048时会出现警告信息，大于2048时，就会报错。
> 注：
> ①一般soft的值会比hard小，也可相等。
> ② /etc/security/limits.d/ 里面配置会覆盖 /etc/security/limits.conf 的配置
> ③只有root用户才有权限修改/etc/security/limits.conf
> ④如果limits.conf没有做设定，则默认值是1024  



##### 8.虚拟内存  

找到文件  

```
vi /etc/sysctl.conf
```

进行设置

```
在文件末尾进行设置
vm.swappiness=1
vm.max_map_count=262144
```

> 验证：  sysctl vm.max_map_count  

如果未生效，使用sysctl -p重新加载配置文件，看文件是否有误，不指定默认加载sysctl.conf  

> sysctl -p

> es使用hybrid max mapfs / niofs目录来存储index数据，操作系统的默认mmap count限制是很
> 低的，可能会导致内存耗尽的异常。
> 需要提升max map count的限制：sysctl -w vm.max_map_count=262144
> 如果要永久性设置这个值，要修改/etc/sysctl.conf，将vm.max_map_count的值修改一下，重启
> 过后，用sysctl vm.max_map_count来验证一下数值是否修改成功
> es同时会用NioFS和MMapFS来处理不同的文件，我们需要设置最大的map count，这样我们才
> 能有足够的虚拟内存来给mmapped文件使用，可以用sysctl来设置：sysctl -w
> vm.max_map_count=262144。还可以再/etc/sysctl.conf中，对vm.max_map_count来设置。

- 禁用所有的swapping file

> 通常来说，es进程会在一个节点上单独运行，那么es进程的内存使用是由jvm option控制
> 的。
> swap（交换分区）主要是在内存不够用的时候，将部分内存上的数据交换到swap空间上，
> 以便让系统不会因内存不够用而导致oom或者更致命的情况出现。
> 我们思考一下如果频繁的将es进程的内存swap到磁盘上，绝对会是一个服务器的性能杀手。想象
> 一下，内存中的操作都是要求快速完成的，如果需要将内存页的数据从磁盘swap回main
> memory的话，性能会有多差。如果内存被swap到了磁盘，那么100微秒的操作会瞬间变成10毫
> 秒，那么如果是大量的这种内存操作呢？这会导致性能急剧下降。
> 可以使用下面的命令临时性禁止swap：swapoff -a
> 要永久性的禁止swap，需要修改/etc/fstab文件，然后将所有包含swap的行都注释掉  



##### 9.配置swappiness  

![image-20201114231252231](https://raw.githubusercontent.com/Yangliang266/images/master/img/image-20201114231252231.png)



> 另外一个方法就是通过sysctl，将vm.swappiness设置为1，这可以尽量减少linux内核swap的倾向，在正常情况下，就不会进行swap，但是在紧急情况下，还是会进行swap操作。 也就是vm.swappiness=1 ，如果你希望完全不会swap。那我们还是需要配置swappiness值为0，那么内存在free和file-backed使用的页面总量小于高水位标记（high water mark也就是我们设置的值）之前，不会发生交换  

