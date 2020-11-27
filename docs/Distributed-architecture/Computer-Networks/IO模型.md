### 一 使用协议进行通信

> tcp 连接建立以后，就可以基于这个连接通道来发送和接受消息了，TCP、UDP 都是在基于 Socket 概念上为某类应用场景而扩展出的传输协议（详见可见tcp/ip章节）

#### 1 socket（具体可见socket章节）

> socket 是一种抽象层，应用程序通过它来发送和接收数据，就像应用程序打开一个文件句柄，把数据读写到磁盘上一样。使用 socket 可以把应用程序添加到网络中，并与处于同一个网络中的其他应用程序进行通信。
>
> socket是在tcp/ip协议族基础上衍生出的编程接口，为了区别于在同一tcp/ip网络协议下的多个应用进程。
>
> socket只是对TCP/IP协议栈操作的抽象，而不是简单的映射关系

#### 2 类型

- 流套接字

- 数据报文套接字

#### 3 socket通信模型

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/socket通信模型.png)



### 二 IO模型

> socket通信模型下衍生出了以下几种io模型，即读写模型

#### 1 同步阻塞io

**BIO**

> 传统java i/o模型 bio 同步阻塞模型

> 即在操作io的同时，进程也会阻塞，直到io操作完毕

#### 2 同步非阻塞io

**NIO**

> nio是一个同步非阻塞io模型

> nio简称new io 是jdk 推出的新的io模型，但具有非阻塞功能更习惯的称为non-blocking io

> 进程发起一个io操作可以去做其他的事情，但是用户进程会一直询问io的处理情况（即io是否就绪），会有一些cpu资源上的浪费

#### 3 异步阻塞io

**AIO 待续**

> 即io操作交由内核操作，具体的操作结果会返回给进程，进程回去负责其他的流程，极大的提高了cpu运行效率

#### 4 异步非阻塞io

1.4 未有





### 二 NIO的三大组件

**buffer 缓冲区**

> 机制

> 1将channel read的操作的数据存入接收缓存区

> 2通过io write将buffer的数据写入到channel

**channel 通道**

> 机制：数据会在channel中传输（channel和socket），通过channel进行读写

> 特点：全双工，双向（读写可以进行）

**selelctor 多路复用器**

> I/O多路复用（multiplexing）的本质是通过一种机制（系统内核缓冲I/O数据），让单个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作

> 此就绪包含tcp连接接入，读和写

> 多路复用机制：提升服务端同时并行处理的连接数量

> select、poll 和 epoll 都是 Linux API 提供的 IO 复用方式。

|            |                       select                       |                       poll                       |                            epoll                             |
| :--------- | :------------------------------------------------: | :----------------------------------------------: | :----------------------------------------------------------: |
| 操作方式   |                        遍历                        |                       遍历                       |                             回调                             |
| 底层实现   |                        数组                        |                       链表                       |                            红黑树                            |
| IO效率     |      每次调用都进行线性遍历，时间复杂度为O(n)      |     每次调用都进行线性遍历，时间复杂度为O(n)     | 事件通知方式，每当fd就绪，系统注册的回调函数就会被调用，将就绪fd放到readyList里面，时间复杂度O(1) |
| 最大连接数 |              1024（x86）或2048（x64）              |                      无上限                      |                            无上限                            |
| fd拷贝     | 每次调用select，都需要把fd集合从用户态拷贝到内核态 | 每次调用poll，都需要把fd集合从用户态拷贝到内核态 |  调用epoll_ctl时拷贝进内核并保存，之后每次epoll_wait不拷贝   |

> epoll是Linux目前大规模网络并发程序开发的首选模型。在绝大多数情况下性能远超select和poll。目前流行的高性能web服务器Nginx正式依赖于epoll提供的高效网络套接字轮询服务。但是，在并发连接不高的情况下，多线程+阻塞I/O方式可能性能更好



> 由于selector一直(轮询,事件通知，具体看采用哪种方式）channel是否就绪，通过判断SelectionKey，这里说明下channel是通过register方法注册到selecor中建立关系，

> 一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系

```java
// 客户端
public class NioClient {

    // (1)创建发送和接受缓冲区
    private static ByteBuffer sendbuffer = ByteBuffer.allocate(1024);
    private static ByteBuffer receivebuffer = ByteBuffer.allocate(1024);

    public static void main(String[] args) throws IOException {
        // (2) 获取一个客户端socket通道
        SocketChannel socketChannel = SocketChannel.open();
        // （3）设置socket为非阻塞方式
        socketChannel.configureBlocking(false);
        // （4）获取一个选择器
        Selector selector = Selector.open();
        // （5）注册客户端socket到选择器
        SelectionKey selectionKey = socketChannel.register(selector, 0);
        // （6）发起连接
        boolean isConnected = socketChannel.connect(new InetSocketAddress("127.0.0.1", 7001));

        // (7)如果连接没有马上建立成功，则设置对链接完成事件感兴趣
        if (!isConnected) {
            selectionKey.interestOps(SelectionKey.OP_CONNECT);

        }

        int num = 0;
        while (true) {

            // (8) 选择已经就绪的网络IO操作，阻塞方法
            int selectCount = selector.select();
            System.out.println(num + "selectCount:" + selectCount);
            // （9）返回已经就绪的通道的事件
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            //(10)处理所有就绪事件
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            SocketChannel client;
            while (iterator.hasNext()) {
                //(10.1)获取一个事件，并从集合移除
                selectionKey = iterator.next();
                iterator.remove();
                //(10.2)获取事件类型
                int readyOps = selectionKey.readyOps();
                //(10.3)判断是否是OP_CONNECT事件
                if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
                    //（10.3.1）等待客户端socket完成与服务器端的链接
                    client = (SocketChannel) selectionKey.channel();
                    if (!client.finishConnect()) {
                        throw new Error();

                    }

                    System.out.println("--- client already connected----");

                    //(10.3.2)设置要发送给服务端的数据
                    sendbuffer.clear();
                    sendbuffer.put("hello server,im a client".getBytes());
                    sendbuffer.flip();
                    //(10.3.3)写入输入。
                    client.write(sendbuffer);
                    //(10.3.4)设置感兴趣事件，读事件
                    selectionKey.interestOps(SelectionKey.OP_READ);

                //(10.4)判断是否是OP_READ事件
                } else if ((readyOps & SelectionKey.OP_READ) != 0) {
                    client = (SocketChannel) selectionKey.channel();
                    //（10.4.1）读取数据并打印
                    receivebuffer.clear();
                    int count = client.read(receivebuffer);
                    if (count > 0) {
                        String temp = new String(receivebuffer.array(), 0, count);
                        System.out.println(num++ + "receive from server:" + temp);
                    }

                }
            }
        }
    }
```



```java
public class NioServer {

    // (1) 缓冲区
    private ByteBuffer sendbuffer = ByteBuffer.allocate(1024);
    private ByteBuffer receivebuffer = ByteBuffer.allocate(1024);
    private Selector selector;

    public NioServer(int port) throws IOException {
        // (2)获取一个服务器套接字通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // （3）socket为非阻塞
        serverSocketChannel.configureBlocking(false);
        // （4）获取与该通道关联的服务端套接字
        ServerSocket serverSocket = serverSocketChannel.socket();
        // （5）绑定服务端地址
        serverSocket.bind(new InetSocketAddress(port));
        // （6）获取一个选择器
        selector = Selector.open();
        // （7）注册通道到选择器，选择对OP_ACCEPT事件感兴趣
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("----Server Started----");

        // (8)处理事件
        int num = 0;
        while (true) {
            // (8.1)获取就绪的事件集合
            int selectKeyCount = selector.select();
            System.out.println(num++ + "selectCount:" + selectKeyCount);

            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            // (8.2)处理就绪事件
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                iterator.remove();
                processSelectedKey(selectionKey);
            }
        }
    }

    private void processSelectedKey(SelectionKey selectionKey) throws IOException {

        SocketChannel client = null;
        // (8.2.1)客户端完成与服务器三次握手
        if (selectionKey.isAcceptable()) {
            // (8.2.1.1)获取完成三次握手的链接套接字
            ServerSocketChannel server = (ServerSocketChannel) selectionKey.channel();
            client = server.accept();
            if (null == client) {
                return;
            }
            System.out.println("--- accepted client---");

            // （8.2.1.2）该套接字为非阻塞模式
            client.configureBlocking(false);
            // （8.2.1.3）注册该套接字到选择器，对OP_READ事件感兴趣
            client.register(selector, SelectionKey.OP_READ);

            // (8.2.2)为读取事件
        } else if (selectionKey.isReadable()) {
            // (8.2.2.1) 读取数据
            client = (SocketChannel) selectionKey.channel();
            receivebuffer.clear();
            int count = client.read(receivebuffer);
            if (count > 0) {
                String receiveContext = new String(receivebuffer.array(), 0, count);
                System.out.println("receive client info:" + receiveContext);
            }
            // (8.2.2.2)发送数据到client
            sendbuffer.clear();
            client = (SocketChannel) selectionKey.channel();
            String sendContent = "hello client ,im server";
            sendbuffer.put(sendContent.getBytes());
            sendbuffer.flip();
            client.write(sendbuffer);
            System.out.println("send info to client:" + sendContent);

        }

    }


    public static void main(String[] args) throws IOException {
        int port = 7001;
        NioServer server = new NioServer(port);
    }
}

其中我们看到8.2.1 看到完成三次握手机制

1 客户端会发送syn = j给服务器 并进入SYN_SEND状态，等待服务器确认
2 服务端接受 确认syn（ack = j+1） 再次发送syn（syn = k）给客户端，此时服务器进入SYN_RECV状态
3 客户端接收到SYN+ACK包，向服务器发送确认包ACK(k+1)此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。
```

