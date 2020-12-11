## Apache与Tomcat

**两者定位：Apache是HTTP Web服务器，Tomcat是Web容器。**

有一个非常形象的比喻：Apache是一辆车，可以装载静态的物件（HTML静态网页等）；但不能装动态的水（JSP、CGI等），要装水就需要桶（容器），当然桶也可以不放在车上而单独存放，则该容器即为Tomcat。

两者的主要区别：

- Apache是世界上最流行的Web服务器（其次是微软的IIS），可以处理浏览器的HTTP请求，默认端口为80；Tomcat是运行在Apache之上的应用服务器，为客户端提供可以调用的方法。Tomcat是一个Servlet容器（可以认为Apache的扩展），可独立运行。
- Apache只支持HTML等静态普通网页，可以单向连通Tomcat（Apache可以访问Tomcat的资源，反之不然）；Tomcat是Servlet容器，可以支持JSP、PHP和CGI等，其中CGI是公共网关接口，可以用Perl编写CGI脚本。
- Apache侧重于HTTP Server；Tomcat侧重于Servlet引擎。

两者可以整合：当客户端需要请求静态资源，只需要Apache服务器响应请求；当客户端需要动态资源，如JSP，需要Tomcat服务容器（Tomcat可以将JSP解析为Servlet）。由于JSP需要JDK的数据库驱动接口，所以一般组合是Apache+Tomcat+JDK。



## Servlet容器（Tomcat）

由于Servlet没有main方法，Servlet生命周期方法的调用受控于容器，即容器管理Servlet的生命周期，包括初始化（init）、服务调用（service）和销毁（destroy），Tomcat就是一个Servlet容器。**负责对象的生命周期的转换和相关服务的连接**。

当Web服务器得到一个Servlet请求时，并不是直接将请求提交给Servlet，而是转交给部署该Servlet的Web容器（Tomcat），由容器向Servlet提供HTTP请求和响应，并由容器调用Servlet的方法，如doGet()和doPost()。

![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/1012728-20170618230610040-1206805475.png)

## Servlet

Servlet没有main方法，不能独立运行，它必须被部署到Servlet容器中，由容器来实例化和调用 Servlet的方法（如doGet()和doPost()），Servlet容器在Servlet的生命周期内包容和管理Servlet。在JSP技术 推出后，管理和运行Servlet/JSP的容器也称为Web容器。