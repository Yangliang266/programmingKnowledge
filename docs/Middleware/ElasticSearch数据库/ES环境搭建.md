## Elasticsearch

#### 1 系统配置修改 

- 最大进程数 修改

  `vi etc/security/limits.config`

  ```shell
  * soft nproc 131072
  * hard nproc 131072
  * soft nofile 131072
  * hard nofile 131072  
  ```

  ```
  设置限制数量，第一列表示用户，* 表示所有用户
  soft xxx : 代表警告的设定，可以超过这个设定值，但是超过后会有警告。
  hard xxx : 代表严格的设定，不允许超过这个设定的值。
  nproc : 是操作系统级别对每个用户创建的进程数的限制
  nofile : 是每个进程可以打开的文件数的限制
  soft nproc ：单个用户可用的最大进程数量(超过会警告);
  hard nproc：单个用户可用的最大进程数量(超过会报错);
  soft nofile ：可打开的文件描述符的最大数(超过会警告);
  hard nofile ：可打开的文件描述符的最大数(超过会报错);
  举例：soft 设为1024，hard设为2048 ，则当你使用数在1~1024之间时可以随便使用，
  1024~2048时会出现警告信息，大于2048时，就会报错。
  注：
  ①一般soft的值会比hard小，也可相等。
  ② /etc/security/limits.d/ 里面配置会覆盖 /etc/security/limits.conf 的配置
  ③只有root用户才有权限修改/etc/security/limits.conf
  ④如果limits.conf没有做设定，则默认值是1024  
  ```

  

- 设置最大缓存

  `vi etc/sysctl.config`

  ```shell
  vm.swappiness=1 //禁用内存和硬盘交换
  vm.max_map_count=262144  //最大虚拟内存
  ```

  `sysctl -p // 生效`



#### 2 启动修改

- 设置es jvm内存

  `vi /es/elasticsearch-7.9.1/config/jvm.options  `

  ```
  在配置文件中设置
  -Xms8g
  -Xmx8g
  根据生产情况设定。建议-Xms与-Xmx配置成一样，同时不要超过32G，一些文档说是30.5G
  ```

- 设置es启动默认使用自身的jdk

  `vi bin/elasticsearch  	`

```shell
#配置ES自带的jdk
export JAVA_HOME=/usr/local/datamn/elasticsearch-7.9.3/jdk
export PATH=$JAVA_HOME/bin:$PATH
#添加jdk判断,注意，要带小引号。
if [ -x "$JAVA_HOME/bin/java" ]; then
JAVA="/usr/local/datamn/elasticsearch-7.9.3/jdk/bin/java"
else
JAVA=`which java`
fi  
```



#### 3 配置文件修改

`vi /es/elasticsearch-7.9.1/config/elasticsearch.yml  `

```shell
#参数解释下面有，这里写不下
network.host: 192.168.88.118
discovery.seed_hosts: ["192.168.88.118:9300"]
cluster.initial_master_nodes: ["192.168.88.118:9300"]  
```

```shell
cluster.name:
配置elasticsearch的集群名称，默认是elasticsearch。建议修改成一个有意义的名称。
node.name:
节点名，通常一台物理服务器就是一个节点，es会默认随机指定一个名字，建议指定一个有意义的名称，方便管理一个或多个节点组成一个
cluster
集群，集群是一个逻辑的概念，节点是物理概念。
path.conf:
设置配置文件的存储路径，tar或zip包安装默认在es根目录下的config文件夹，rpm安装默认在/etc/ elasticsearch
path.data:
设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开。
path.logs:
设置日志文件的存储路径，默认是es根目录下的logs文件夹
path.plugins:
设置插件的存放路径，默认是es根目录下的plugins文件夹
bootstrap.memory_lock: true
设置为true可以锁住ES使用的内存，避免内存与swap分区交换数据。
network.host:
设置绑定主机的ip地址，设置为0.0.0.0表示绑定任何ip，允许外网访问，生产环境建议设置为具体的ip。
http.port: 9200
设置对外服务的http端口，默认为9200。
transport.tcp.port: 9300 
集群结点之间通信端口
node.master:
指定该节点是否有资格被选举成为master结点，默认是true，如果原来的master宕机会重新选举新的master。
node.data:
指定该节点是否存储索引数据，默认为true。
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
es7.x 之后新增的配置，节点发现
cluster.initial_master_nodes: ["node1", "node2","node3"]
es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
node.max_local_storage_nodes:
单机允许的最大存储结点数，通常单机启动一个结点建议设置为1，开发环境如果单机启动多个节点可设置大于1。  
```



#### 4 创建新用户

`useradd es`

root 用户下赋予es权限

`chown -R es:es /usr/local/datamn/elasticsearch-7.9.3`



#### 5 切换用户启动

`su es`

`./bin/elasticsearch`





### kibana

```
# vi /etc/kibana/kibana.yml
修改配置
server.port: 5601 #这个不一定需要配置
server.host: "192.168.56.12" 配置IP
elasticsearch.hosts: ["http://192.168.56.15:9200"]
i18n.locale: "zh-CN" #默认是英文，这个是中文设置
```

在root下为 为kibana赋权

`chown -R es:es /usr/local/datamn/kibana-7.9.3-linux-x86_64/ `

启动kibana

`./bin/kibana`

