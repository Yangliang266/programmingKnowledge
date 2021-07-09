## 一 什么是Tomcat

> Tomcat软件是Java Servlet的开源实现，JavaServer Pages, JavaExpression Language 和Java WebSocket技术
>
> 可以看作是web容器或者servlet容器

### 1 什么是servlet

> java 为了提供web服务功能，制定了servlet规范，tomcat支持了servlet规范 tomcat/lib/servlet-api.jar  

```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    ServletConfig getServletConfig();
    void service(ServletRequest req, ServletResponse res）throws ServletException,IOException;
    String getServletInfo();
    void destroy();
}
                 
class LoginServlet extends HttpServlet{
    doGet(request,response){}
    doPost(request,response){}
}
                 
<servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.gupao.web.servlet.LoginServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/login</url-pattern>
</servlet-mapping>                 
```



### 2 什么是servlet容器和web容器

> servlet容器是一个能够驱使所有servlet运行的一个承载容器
>
> web容器负责socket的创建，端口的监听，请求的接受



![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/clipboard.png)



### 3 tomcat目录结构

（1）bin：主要用来存放命令，.bat是windows下，.sh是Linux下

（2）conf：主要用来存放tomcat的一些配置文件

（3）lib：存放tomcat依赖的一些jar包

（4）logs：存放tomcat在运行时产生的日志文件

（5）temp：存放运行时产生的临时文件

（6）webapps：存放应用程序

（7）work：存放tomcat运行时编译后的文件，比如JSP编译后的文件





### 二 tomcat架构图

![](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/image-20210311100823339.png)



#### 2.1 主要组件说明

##### 2.1.1 server

> 服务器代表整个容器。Tomcat提供了一个服务器接口的默认实现，很少由用户自定义。

##### 2.1.2 service

> 服务是一个中间组件，它位于服务器内部并绑定一个或多个更多的连接器到一个引擎。服务元素很少由用户，作为默认的实现简单而充分:服务interface.Engine

##### 2.1.3 connector

> 连接器处理与客户机的通信。有多个连接器使用Tomcat。其中包括用于大多数HTTP的HTTP连接器流量，特别是在将Tomcat作为独立服务器和AJP连接器运行时哪个实现了将Tomcat连接到诸如此类的web服务器时使用的AJP协议Apache HTTPD服务器。创建自定义连接器是一项重要的工作

##### 2.1.4 engine

> 引擎表示特定服务的请求处理管道。作为服务可能有多个连接器，引擎接收和处理所有请求将响应返回给相应的连接器传输到客户端。可以实现引擎接口来提供定制引擎，虽然这是不常见的。注意，该引擎可以通过jvmRoute用于Tomcat服务器集群参数

##### 2.1.5 host

> 主机是一个网络名称(如www.yourcompany.com)与Tomcat的关联服务器。一个引擎可以包含多个主机，并且主机元素也支持网络别名，如yourcompany.com  和abc.yourcompany.com。用户很少创建自定义主机，因为标准主机实现提供了重要的附加功能功能

##### 2.1.6 context

> 上下文表示web应用程序。一个主机可以包含多个上下文有唯一的路径。可以实现上下文接口来创建自定义但是这种情况很少发生，因为StandardContext提供了重要的信息额外的功能



#### 2.2 组件分类

##### connector 既可向外，又可向内

> Endpoint接收Socket连接，生成一个SocketProcessor任务提交到线程池去处理SocketProcessor的run方法会调用Processor组件去解析应用层协议，Processor通过解析生成Request对象后，会调用Adapter的service方法  

![image-20210311101738604](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/Javaweb/tomcat/image-20210311101738604.png)

###### EndPoint
> 监听通信端口，是对传输层的抽象，用来实现 TCP/IP 协议的。
> 对应的抽象类为AbstractEndPoint，有很多实现类，比如NioEndPoint，JIoEndPoint等。在其中有两个组件，一个是Acceptor，另外一个是SocketProcessor。
> Acceptor用于监听Socket连接请求，SocketProcessor用于处理接收到的Socket请求。

###### Processor
> Processor是用于实现HTTP协议的，也就是说Processor是针对应用层协议的抽象。
> Processor接受来自EndPoint的Socket，然后解析成Tomcat Request和Tomcat Response对象，最后通过Adapter提交给容器。
> 对应的抽象类为AbstractProcessor，有很多实现类，比如AjpProcessor、Http11Processor等。

###### Adpater
> ProtocolHandler接口负责解析请求并生成 Tomcat Request 类。需要把这个 Request 对象转换成 ServletRequest。
> Tomcat 引入CoyoteAdapter，这是**适配器模式**的经典运用，连接器调用 CoyoteAdapter 的 service 方法，传入的是Tomcat Request 对象，CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 service 方法。  



##### container 对内

> 通过Connector之后，已经能够获得对应的Servlet



#### 2.3 自定义类加载器

WebAppClassLoader，打破了双亲委派模型：先自己尝试去加载这个类，找不到再委托给父类加载器。

通过复写findClass和loadClass实现。



#### 2.4 Session管理

Context中的Manager对象，查看Manager中的方法，可以发现有跟Session相关的。

Session 的核心原理是通过 Filter 拦截 Servlet 请求，将标准的 ServletRequest 包装一下，换成 Spring 的Request 对象，这样当我们调用 Request 对象的 getSession 方法时，Spring 在背后为我们创建和管理Session。





### 三 源码分析

#### BootStrap

##### BootStrap是Tomcat的入口类

Endpoint接收Socket连接，生成一个SocketProcessor任务提交到线程池去处理

SocketProcessor的run方法会调用Processor组件去解析应用层协议，Processor通过解析生成Request对象后，会调用Adapter的service方法。bootstrap.init() 和 创建Catalina

##### Catalina

解析server.xml文件

创建Server组件，并且调用其init和start方法

##### Lifecycle

用于管理各个组件的生命周期

init、start、stop、destroy

LifecycleBase实现了Lifecycle，利用的是模板设计模式

##### Server

管理Service组件，并且调用其init和start方法

##### Service

管理连接器和Engine



