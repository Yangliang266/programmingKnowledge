zookeeper处理流程

1.  main方法，加载zoo.cfg

2.  解析配置文件

3.  网络通信

4. 1. 选举，即多个zk节点的数据通信
   2. 客户端请求，即客户端要连接的zkserver
   3. 数据同步，即leader和follow节点之间的数据同步

5.  功能层面

6. 1. 数据的存储，持久化
   2. 数据同步
   3. watch机制

1 main 方法加载zoo.cfg

```java
public static void main(String[] args) {
    QuorumPeerMain main = new QuorumPeerMain();
    try {
        //1 zoo.cfg 
        main.initializeAndRun(args);
    } catch (IllegalArgumentException e) {
        LOG.error("Invalid arguments, exiting abnormally", e);
        LOG.info(USAGE);
        System.err.println(USAGE);
        System.exit(2);
    } catch (ConfigException e) {
        LOG.error("Invalid config, exiting abnormally", e);
        System.err.println("Invalid config, exiting abnormally");
        System.exit(2);
    } catch (Exception e) {
        LOG.error("Unexpected exception, exiting abnormally", e);
        System.exit(1);
    }
    LOG.info("Exiting normally");
    System.exit(0);
}
```

2 初始化

```java
protected void initializeAndRun(String[] args)
    throws ConfigException, IOException
{
    //保存zoo.cfg文件解析之后的所有参数（一定在后面有用到）
    QuorumPeerConfig config = new QuorumPeerConfig();
    if (args.length == 1) {
        config.parse(args[0]); // 2 解析
    }

    // ()Start and schedule the the purge task
    DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config
            .getDataDir(), config.getDataLogDir(), config
            .getSnapRetainCount(), config.getPurgeInterval());
    purgeMgr.start();

    //3.1 判断是单点
    if (args.length == 1 && config.servers.size() > 0) {
        runFromConfig(config);//如果args==1，走这段代码
    } else {
    // 3.2 判断是集群 
        LOG.warn("Either no config or no quorum defined in config, running "
                + " in standalone mode");
        // there is only server in the quorum -- run as standalone
        ZooKeeperServerMain.main(args);
    }
}
```

3.1 单点模式流程

```java
public void runFromConfig(QuorumPeerConfig config) throws IOException {
  try {
      ManagedUtil.registerLog4jMBeans();
  } catch (JMException e) {
      LOG.warn("Unable to register log4j JMX control", e);
  }

  LOG.info("Starting quorum peer");
  try {
      // 3.1.2.1 创建通信工厂，和通信有关，默认走nio， 可以手动配置
      ServerCnxnFactory cnxnFactory = ServerCnxnFactory.createFactory();
      // 3.1.2.2 nio初始配置
      cnxnFactory.configure(config.getClientPortAddress(),
                            config.getMaxClientCnxns());

      quorumPeer = getQuorumPeer();
      //getView()
      quorumPeer.setQuorumPeers(config.getServers()); //zoo.cfg里面解析的servers节点
      quorumPeer.setTxnFactory(new FileTxnSnapLog(
              new File(config.getDataLogDir()),
              new File(config.getDataDir())));
      quorumPeer.setElectionType(config.getElectionAlg());
      quorumPeer.setMyid(config.getServerId());
      quorumPeer.setTickTime(config.getTickTime());
      quorumPeer.setInitLimit(config.getInitLimit());
      quorumPeer.setSyncLimit(config.getSyncLimit());
      quorumPeer.setQuorumListenOnAllIPs(config.getQuorumListenOnAllIPs());
      quorumPeer.setCnxnFactory(cnxnFactory);// 设置cnxnFacotory
      quorumPeer.setQuorumVerifier(config.getQuorumVerifier());
      quorumPeer.setClientPortAddress(config.getClientPortAddress());
      quorumPeer.setMinSessionTimeout(config.getMinSessionTimeout());
      quorumPeer.setMaxSessionTimeout(config.getMaxSessionTimeout());
      quorumPeer.setZKDatabase(new ZKDatabase(quorumPeer.getTxnFactory()));
      quorumPeer.setLearnerType(config.getPeerType());
      quorumPeer.setSyncEnabled(config.getSyncEnabled());

      // sets quorum sasl authentication configurations
      quorumPeer.setQuorumSaslEnabled(config.quorumEnableSasl);
      if(quorumPeer.isQuorumSaslAuthEnabled()){
          quorumPeer.setQuorumServerSaslRequired(config.quorumServerRequireSasl);
          quorumPeer.setQuorumLearnerSaslRequired(config.quorumLearnerRequireSasl);
          quorumPeer.setQuorumServicePrincipal(config.quorumServicePrincipal);
          quorumPeer.setQuorumServerLoginContext(config.quorumServerLoginContext);
          quorumPeer.setQuorumLearnerLoginContext(config.quorumLearnerLoginContext);
      }

      quorumPeer.setQuorumCnxnThreadsSize(config.quorumCnxnThreadsSize);
      quorumPeer.initialize();

      quorumPeer.start(); // 3.1 重点 ： 单点启动
      quorumPeer.join();
  } catch (InterruptedException e) {
      // warn, but generally this is ok
      LOG.warn("Quorum Peer interrupted", e);
  }
}
```

3.1 单点启动

```java
public synchronized void start() {
    // 3.1.1
    loadDataBase(); //加载数据()
    // 3.1.2
    cnxnFactory.start();      //cnxnFacotory?  跟通信有关系. ->暴露一个2181的端口号
    // 3.1.3
    startLeaderElection();  //开始leader选举-> 启动一个投票的监听、初始化一个选举算法FastLeader.
    // 3.1.4
    super.start(); //当前的QuorumPeer继承Thread，调用Thread.start() ->QuorumPeer.run() 有多种实现
}
```

3.1.2 通信启动

```java
public void start() {
    // ensure thread is started once and only once
    if (thread.getState() == Thread.State.NEW) {
        thread.start(); // 3.1.2.1 当前构建的zookeeper的thread
    }
}
```

3.1.2.1 当前构建的zookeeper的thread

3.1.2.2 nio初始配置nio初始配置

```java
Thread thread;
@Override
public void configure(InetSocketAddress addr, int maxcc) throws IOException {
    configureSaslLogin();

    thread = new ZooKeeperThread(this, "NIOServerCxn.Factory:" + addr);
    thread.setDaemon(true);
    maxClientCnxns = maxcc;
    this.ss = ServerSocketChannel.open();
    ss.socket().setReuseAddress(true);//端口可复用
    LOG.info("binding to port " + addr);
    ss.socket().bind(addr); //绑定ip和端口
    ss.configureBlocking(false); //非阻塞
    ss.register(selector, SelectionKey.OP_ACCEPT); //注册一个accept
} 	
```



```java
//当收到客户端的请求时，会需要从这个方法里面来看-> create/delete/setdata
public void run() {
    while (!ss.socket().isClosed()) {
        try {
            selector.select(1000);
            Set<SelectionKey> selected;
            synchronized (this) {
                selected = selector.selectedKeys();
            }
            ArrayList<SelectionKey> selectedList = new ArrayList<SelectionKey>(
                    selected);
            Collections.shuffle(selectedList);
            for (SelectionKey k : selectedList) {
                if ((k.readyOps() & SelectionKey.OP_ACCEPT) != 0) {
                    SocketChannel sc = ((ServerSocketChannel) k
                            .channel()).accept();
                    InetAddress ia = sc.socket().getInetAddress();
                    int cnxncount = getClientCnxnCount(ia);
                    if (maxClientCnxns > 0 && cnxncount >= maxClientCnxns){
                        LOG.warn("Too many connections from " + ia
                                 + " - max is " + maxClientCnxns );
                        sc.close();
                    } else {
                        LOG.info("Accepted socket connection from "
                                 + sc.socket().getRemoteSocketAddress());
                        sc.configureBlocking(false);
                        SelectionKey sk = sc.register(selector,
                                SelectionKey.OP_READ);
                        NIOServerCnxn cnxn = createConnection(sc, sk);
                        sk.attach(cnxn);
                        addCnxn(cnxn);
                    }
                } else if ((k.readyOps() & (SelectionKey.OP_READ | SelectionKey.OP_WRITE)) != 0) {
                    NIOServerCnxn c = (NIOServerCnxn) k.attachment();
                    c.doIO(k);
                } else {
                    if (LOG.isDebugEnabled()) {
                        LOG.debug("Unexpected ops in select "
                                  + k.readyOps());
                    }
                }
            }
            selected.clear();
        } catch (RuntimeException e) {
            LOG.warn("Ignoring unexpected runtime exception", e);
        } catch (Exception e) {
            LOG.warn("Ignoring exception", e);
        }
    }
    closeAll();
    LOG.info("NIOServerCnxn factory exited run method");
}
```



3.1.3 开始leader选举，开启投票监听和选举算法

```java
synchronized public void startLeaderElection() {
   try {
       //构建一个票据， （myid ,zxid ,epoch）,用来投票的。
      currentVote = new Vote(myid, getLastLoggedZxid(), getCurrentEpoch());
   } catch(IOException e) {
      RuntimeException re = new RuntimeException(e.getMessage());
      re.setStackTrace(e.getStackTrace());
      throw re;
   }
   // 判断节点是否在同一个视图中
    for (QuorumServer p : getView().values()) {
        if (p.id == myid) {
            myQuorumAddr = p.addr; //地址： 1 ->myQuorumAddr=192.168.13.102
            break;
        }
    }
    if (myQuorumAddr == null) {
        throw new RuntimeException("My id " + myid + " not in the peer list");
    }
    if (electionType == 0) { //选举的策略
        try {
            udpSocket = new DatagramSocket(myQuorumAddr.getPort());
            responder = new ResponderThread();
            responder.start();
        } catch (SocketException e) {
            throw new RuntimeException(e);
        }
    }
    // 3.1.3.1
    this.electionAlg = createElectionAlgorithm(electionType);
}
```

3.1.3.1 开启投票监听

3.1.3.2 选举算法

```java
protected Election createElectionAlgorithm(int electionAlgorithm){
    Election le=null;
            
    //TODO: use a factory rather than a switch
    switch (electionAlgorithm) {
    case 0:
        le = new LeaderElection(this);
        break;
    case 1:
        le = new AuthFastLeaderElection(this);
        break;
    case 2:
        le = new AuthFastLeaderElection(this, true);
        break;
    case 3:
        //跟选举有关系，用来接收投票的。
        qcm = createCnxnManager();
        // 监听相同端口号的票据
        QuorumCnxManager.Listener listener = qcm.listener;
        if(listener != null){
            listener.start();
            //创建一个FastLeaderElection选举算法
            le = new FastLeaderElection(this, qcm);
        } else {
            LOG.error("Null listener when initializing cnx manager");
        }
        break;
    default:
        assert false;
    }
    return le;
}
```

3.1.3.2 选举算法

```java
public FastLeaderElection(QuorumPeer self, QuorumCnxManager manager){
    this.stop = false;
    this.manager = manager; // 票据通信
    // 3.1.3.2 阻塞队列，存放接受票据，和发送票据
    starter(self, manager);
}
```



3.1.3.2.1 初始化队列

```java
private void starter(QuorumPeer self, QuorumCnxManager manager) {
    this.self = self;
    proposedLeader = -1;
    proposedZxid = -1;

    sendqueue = new LinkedBlockingQueue<ToSend>();
    recvqueue = new LinkedBlockingQueue<Notification>();
    this.messenger = new Messenger(manager);
}
```

3.1.3.2.2  投票通信

```java
Messenger(QuorumCnxManager manager) {

    this.ws = new WorkerSender(manager); //发送票据的线程（用于消费sendQueue）

    Thread t = new Thread(this.ws,
            "WorkerSender[myid=" + self.getId() + "]");
    t.setDaemon(true); //守护线程
    t.start();

    this.wr = new WorkerReceiver(manager);//接收票据的线程

    t = new Thread(this.wr,
            "WorkerReceiver[myid=" + self.getId() + "]");
    t.setDaemon(true);
    t.start();
}
```

3.1.3.1 监听相同端口的票据

```java
/**
 * Thread to listen on some port
 */
public class Listener extends ZooKeeperThread {

    volatile ServerSocket ss = null;

    public Listener() {
        // During startup of thread, thread name will be overridden to
        // specific election address
        super("ListenerThread");
    }

    /**
     * Sleeps on accept().
     *TODO -> 接收投票的处理（假设）
     */
    @Override
    public void run() {
        int numRetries = 0;
        InetSocketAddress addr;
        while((!shutdown) && (numRetries < 3)){
            try {
                ss = new ServerSocket(); //不是NIO， socket io
                ss.setReuseAddress(true);
                if (listenOnAllIPs) {
                    int port = view.get(QuorumCnxManager.this.mySid)
                        .electionAddr.getPort();
                    addr = new InetSocketAddress(port);
                } else {
                    addr = view.get(QuorumCnxManager.this.mySid)
                        .electionAddr;
                }
                LOG.info("My election bind port: " + addr.toString());
                setName(view.get(QuorumCnxManager.this.mySid)
                        .electionAddr.toString());
                ss.bind(addr);
                while (!shutdown) {
                    Socket client = ss.accept();
                    setSockOpts(client);
                    LOG.info("Received connection request "
                            + client.getRemoteSocketAddress());

                    // Receive and handle the connection request
                    // asynchronously if the quorum sasl authentication is
                    // enabled. This is required because sasl server
                    // authentication process may take few seconds to finish,
                    // this may delay next peer connection requests.
                    if (quorumSaslAuthEnabled) {
                        receiveConnectionAsync(client);
                    } else {
                        receiveConnection(client);
                    }

                    numRetries = 0;
                }
            } catch (IOException e) {
                LOG.error("Exception while listening", e);
                numRetries++;
                try {
                    ss.close();
                    Thread.sleep(1000);
                } catch (IOException ie) {
                    LOG.error("Error closing server socket", ie);
                } catch (InterruptedException ie) {
                    LOG.error("Interrupted while sleeping. " +
                              "Ignoring exception", ie);
                }
            }
        }
        LOG.info("Leaving listener");
        if (!shutdown) {
            LOG.error("As I'm leaving the listener thread, "
                    + "I won't be able to participate in leader "
                    + "election any longer: "
                    + view.get(QuorumCnxManager.this.mySid).electionAddr);
        }
    }
    
    /**
     * Halts this listener thread.
     */
    void halt(){
        try{
            LOG.debug("Trying to close listener: " + ss);
            if(ss != null) {
                LOG.debug("Closing listener: "
                          + QuorumCnxManager.this.mySid);
                ss.close();
            }
        } catch (IOException e){
            LOG.warn("Exception when shutting down listener: " + e);
        }
    }
}
```

3.1.4 通过通信收集的票据判断谁是leader

```java
public void run() {
    setName("QuorumPeer" + "[myid=" + getId() + "]" +
            cnxnFactory.getLocalAddress());

    LOG.debug("Starting quorum peer");
    try {
        jmxQuorumBean = new QuorumBean(this);
        MBeanRegistry.getInstance().register(jmxQuorumBean, null);
        for(QuorumServer s: getView().values()){
            ZKMBeanInfo p;
            if (getId() == s.id) {
                p = jmxLocalPeerBean = new LocalPeerBean(this);
                try {
                    MBeanRegistry.getInstance().register(p, jmxQuorumBean);
                } catch (Exception e) {
                    LOG.warn("Failed to register with JMX", e);
                    jmxLocalPeerBean = null;
                }
            } else {
                p = new RemotePeerBean(s);
                try {
                    MBeanRegistry.getInstance().register(p, jmxQuorumBean);
                } catch (Exception e) {
                    LOG.warn("Failed to register with JMX", e);
                }
            }
        }
    } catch (Exception e) {
        LOG.warn("Failed to register with JMX", e);
        jmxQuorumBean = null;
    }

    try {
        /*
         * Main loop
         * 死循环
         */
        while (running) {
            switch (getPeerState()) {//第一次启动的时候，LOOKING
            case LOOKING:
                LOG.info("LOOKING");

                if (Boolean.getBoolean("readonlymode.enabled")) {
                    LOG.info("Attempting to start ReadOnlyZooKeeperServer");

                    // Create read-only server but don't start it immediately
                    final ReadOnlyZooKeeperServer roZk = new ReadOnlyZooKeeperServer(
                            logFactory, this,
                            new ZooKeeperServer.BasicDataTreeBuilder(),
                            this.zkDb);

                    // Instead of starting roZk immediately, wait some grace
                    // period before we decide we're partitioned.
                    //
                    // Thread is used here because otherwise it would require
                    // changes in each of election strategy classes which is
                    // unnecessary code coupling.
                    Thread roZkMgr = new Thread() {
                        public void run() {
                            try {
                                // lower-bound grace period to 2 secs
                                sleep(Math.max(2000, tickTime));
                                if (ServerState.LOOKING.equals(getPeerState())) {
                                    roZk.startup();
                                }
                            } catch (InterruptedException e) {
                                LOG.info("Interrupted while attempting to start ReadOnlyZooKeeperServer, not started");
                            } catch (Exception e) {
                                LOG.error("FAILED to start ReadOnlyZooKeeperServer", e);
                            }
                        }
                    };
                    try {
                        roZkMgr.start();
                        setBCVote(null);
                        setCurrentVote(makeLEStrategy().lookForLeader());
                    } catch (Exception e) {
                        LOG.warn("Unexpected exception",e);
                        setPeerState(ServerState.LOOKING);
                    } finally {
                        // If the thread is in the the grace period, interrupt
                        // to come out of waiting.
                        roZkMgr.interrupt();
                        roZk.shutdown();
                    }
                } else {
                    try {
                        setBCVote(null);
                        //
                        // setCurrentVote -> 确定了谁是leader了********************************************
                        setCurrentVote(makeLEStrategy().lookForLeader());
                    } catch (Exception e) {
                        LOG.warn("Unexpected exception", e);
                        setPeerState(ServerState.LOOKING);
                    }
                }
                break;
            case OBSERVING:
                try {
                    LOG.info("OBSERVING");
                    setObserver(makeObserver(logFactory));
                    observer.observeLeader();
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e );                        
                } finally {
                    observer.shutdown();
                    setObserver(null);
                    setPeerState(ServerState.LOOKING);
                }
                break;
            case FOLLOWING:
                try {
                    LOG.info("FOLLOWING");
                    setFollower(makeFollower(logFactory));
                    follower.followLeader(); //连接到leader
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e);
                } finally {
                    follower.shutdown();
                    setFollower(null);
                    setPeerState(ServerState.LOOKING);
                }
                break;
            case LEADING:
                LOG.info("LEADING");
                try {
                    setLeader(makeLeader(logFactory));
                    leader.lead(); //lead 状态
                    setLeader(null);
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e);
                } finally {
                    if (leader != null) {
                        leader.shutdown("Forcing shutdown");
                        setLeader(null);
                    }
                    setPeerState(ServerState.LOOKING);
                }
                break;
            }
        }
    } finally {
        LOG.warn("QuorumPeer main thread exited");
        try {
            MBeanRegistry.getInstance().unregisterAll();
        } catch (Exception e) {
            LOG.warn("Failed to unregister with JMX", e);
        }
        jmxQuorumBean = null;
        jmxLocalPeerBean = null;
    }
}
```

3.1.4.1 具体判断方法，以及最终决策

```java
/**
 * Starts a new round of leader election. Whenever our QuorumPeer
 * changes its state to LOOKING, this method is invoked, and it
 * sends notifications to all other peers.
 */
public Vote lookForLeader() throws InterruptedException {
    try {
        self.jmxLeaderElectionBean = new LeaderElectionBean();
        MBeanRegistry.getInstance().register(
                self.jmxLeaderElectionBean, self.jmxLocalPeerBean);
    } catch (Exception e) {
        LOG.warn("Failed to register with JMX", e);
        self.jmxLeaderElectionBean = null;
    }
    if (self.start_fle == 0) {
       self.start_fle = Time.currentElapsedTime();
    }
    try {
        //接收到的票据的集合
        HashMap<Long, Vote> recvset = new HashMap<Long, Vote>();

        //
        HashMap<Long, Vote> outofelection = new HashMap<Long, Vote>();

        int notTimeout = finalizeWait;

        synchronized(this){
            //逻辑时钟->epoch
            logicalclock.incrementAndGet();
            //proposal
            updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
        }

        LOG.info("New election. My id =  " + self.getId() +
                ", proposed zxid=0x" + Long.toHexString(proposedZxid));
        sendNotifications();//我要广播自己的票据

        /*
         * Loop in which we exchange notifications until we find a leader
         */

        //接收到了票据
        while ((self.getPeerState() == ServerState.LOOKING) &&
                (!stop)){
            /*
             * Remove next notification from queue, times out after 2 times
             * the termination time
             */
            //recvqueue是从网络上接收到的其他机器的Notification
            Notification n = recvqueue.poll(notTimeout,
                    TimeUnit.MILLISECONDS);

            /*
             * Sends more notifications if haven't received enough.
             * Otherwise processes new notification.
             */
            if(n == null){
                if(manager.haveDelivered()){
                    sendNotifications();
                } else {
                    manager.connectAll();//重新连接集群中的所有节点
                }

                /*
                 * Exponential backoff
                 */
                int tmpTimeOut = notTimeout*2;
                notTimeout = (tmpTimeOut < maxNotificationInterval?
                        tmpTimeOut : maxNotificationInterval);
                LOG.info("Notification time out: " + notTimeout);
            }

            else if(validVoter(n.sid) && validVoter(n.leader)) {//判断是否是一个有效的票据
                /*
                 * Only proceed if the vote comes from a replica in the
                 * voting view for a replica in the voting view.
                 */
                switch (n.state) {
                case LOOKING: //第一次进入到这个case
                    // If notification > current, replace and send messages out
                    if (n.electionEpoch > logicalclock.get()) { //
                        logicalclock.set(n.electionEpoch);
                        recvset.clear();//清空
                        //收到票据之后，当前的server要听谁的。
                        //可能是听server1的、也可能是听server2，也可能是听server3
                        //zab  leader选举算法
                        if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                getInitId(), getInitLastLoggedZxid(), getPeerEpoch())) {
                            //把自己的票据更新成对方的票据，那么下一次，发送的票据就是新的票据
                            updateProposal(n.leader, n.zxid, n.peerEpoch);
                        } else {
                            //收到的票据小于当前的节点的票据，下一次发送票据，仍然发送自己的
                            updateProposal(getInitId(),
                                    getInitLastLoggedZxid(),
                                    getPeerEpoch());
                        }
                        //继续发送通知
                        sendNotifications();
                    } else if (n.electionEpoch < logicalclock.get()) { //说明当前的数据已经过期了
                        if(LOG.isDebugEnabled()){
                            LOG.debug("Notification election epoch is smaller than logicalclock. n.electionEpoch = 0x"
                                    + Long.toHexString(n.electionEpoch)
                                    + ", logicalclock=0x" + Long.toHexString(logicalclock.get()));
                        }
                        break;
                    } else if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                            proposedLeader, proposedZxid, proposedEpoch)) {
                        updateProposal(n.leader, n.zxid, n.peerEpoch);
                        sendNotifications();
                    }

                    if(LOG.isDebugEnabled()){
                        LOG.debug("Adding vote: from=" + n.sid +
                                ", proposed leader=" + n.leader +
                                ", proposed zxid=0x" + Long.toHexString(n.zxid) +
                                ", proposed election epoch=0x" + Long.toHexString(n.electionEpoch));
                    }

                    recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));
                    //决断时刻（当前节点的更新后的vote信息，和recvset集合中的票据进行归纳，）
                    if (termPredicate(recvset,
                            new Vote(proposedLeader, proposedZxid,
                                    logicalclock.get(), proposedEpoch))) {

                        // Verify if there is any change in the proposed leader
                        while((n = recvqueue.poll(finalizeWait,
                                TimeUnit.MILLISECONDS)) != null){
                            if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                    proposedLeader, proposedZxid, proposedEpoch)){
                                recvqueue.put(n);
                                break;
                            }
                        }

                        /*
                         * This predicate is true once we don't read any new
                         * relevant message from the reception queue
                         */
                        if (n == null) {
                            self.setPeerState((proposedLeader == self.getId()) ?
                                    ServerState.LEADING: learningState());

                            Vote endVote = new Vote(proposedLeader,
                                                    proposedZxid,
                                                    logicalclock.get(),
                                                    proposedEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                    }
                    break;
                case OBSERVING:
                    LOG.debug("Notification from observer: " + n.sid);
                    break;
                case FOLLOWING:
                case LEADING:
                    /*
                     * Consider all notifications from the same epoch
                     * together.
                     */
                    if(n.electionEpoch == logicalclock.get()){
                        recvset.put(n.sid, new Vote(n.leader,
                                                      n.zxid,
                                                      n.electionEpoch,
                                                      n.peerEpoch));
                       
                        if(ooePredicate(recvset, outofelection, n)) {
                            self.setPeerState((n.leader == self.getId()) ?
                                    ServerState.LEADING: learningState());

                            Vote endVote = new Vote(n.leader, 
                                    n.zxid, 
                                    n.electionEpoch, 
                                    n.peerEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                    }

                    /*
                     * Before joining an established ensemble, verify
                     * a majority is following the same leader.
                     */
                    outofelection.put(n.sid, new Vote(n.version,
                                                        n.leader,
                                                        n.zxid,
                                                        n.electionEpoch,
                                                        n.peerEpoch,
                                                        n.state));
       
                    if(ooePredicate(outofelection, outofelection, n)) {
                        synchronized(this){
                            logicalclock.set(n.electionEpoch);
                            self.setPeerState((n.leader == self.getId()) ?
                                    ServerState.LEADING: learningState());
                        }
                        Vote endVote = new Vote(n.leader,
                                                n.zxid,
                                                n.electionEpoch,
                                                n.peerEpoch);
                        leaveInstance(endVote);
                        return endVote;
                    }
                    break;
                default:
                    LOG.warn("Notification state unrecognized: {} (n.state), {} (n.sid)",
                            n.state, n.sid);
                    break;
                }
            } else {
                if (!validVoter(n.leader)) {
                    LOG.warn("Ignoring notification for non-cluster member sid {} from sid {}", n.leader, n.sid);
                }
                if (!validVoter(n.sid)) {
                    LOG.warn("Ignoring notification for sid {} from non-quorum member sid {}", n.leader, n.sid);
                }
            }
        }
        return null;
    } finally {
        try {
            if(self.jmxLeaderElectionBean != null){
                MBeanRegistry.getInstance().unregister(
                        self.jmxLeaderElectionBean);
            }
        } catch (Exception e) {
            LOG.warn("Failed to unregister with JMX", e);
        }
        self.jmxLeaderElectionBean = null;
        LOG.debug("Number of connection processing threads: {}",
                manager.getConnectionThreadCount());
    }
}
```

广播自己的票据

```java
private void sendNotifications() {
    for (QuorumServer server : self.getVotingView().values()) {
        long sid = server.id;

        ToSend notmsg = new ToSend(ToSend.mType.notification,
                proposedLeader, //myid
                proposedZxid, //zxid
                logicalclock.get(),//epoch
                QuorumPeer.ServerState.LOOKING,//
                sid, //myid
                proposedEpoch); //发起票据epoch
        if(LOG.isDebugEnabled()){
            LOG.debug("Sending Notification: " + proposedLeader + " (n.leader), 0x"  +
                  Long.toHexString(proposedZxid) + " (n.zxid), 0x" + Long.toHexString(logicalclock.get())  +
                  " (n.round), " + sid + " (recipient), " + self.getId() +
                  " (myid), 0x" + Long.toHexString(proposedEpoch) + " (n.peerEpoch)");
        }
        sendqueue.offer(notmsg); //阻塞队列,  线程->生产者消费者模式
    }
}
```

![img](C:\Users\xiaomi\AppData\Local\YNote\data\yangl546493589@163.com\6221faa0252a4af3906a298c4cbdc340\clipboard.png)

| ToSenderleader； 被推荐的服务器 sidzxid; 被推荐的服务器当前最新的事务 idpeerEpoch; 被推荐的服务器当前所处的 epochelectionepoch； 当前服务器所处的 epochstat 当前服务器状态sid 接收消息的服务器 sid（myid） | Notificationleader； //被推荐的服务器 sidzxid； 被推荐的服务器最新事务 idpeerEpoch; 被推荐的服务器当前所处的 epochelectionEpoch 选举服务器所处的 epochstat； 选举服务器当前的状态sid； 选举服务器的 sid |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |