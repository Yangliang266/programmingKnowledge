### 什么是springboot

1. 服务于spring框架的框架
2. 约定优于配置下的产物



#### 约定优于配置的体现

1. maven 的目录结构
   a) 默认有 resources 文件夹存放配置文件
   b) 默认打包方式为 jar
2. spring-boot-starter-web 中默认包含 spring mvc 相关依赖以及内置的 tomcat 容器，使得构建一个 web 应用更加简单
3. 默认提供 application.properties/yml 文件
4. 默认通过 spring.profiles.active 属性来决定运行环境时读取的配置文件
5. EnableAutoConfiguration 默认对于依赖的 starter 进行自动装配
6. 自己理解：规定在什么地方做什么事情，遵循此约定。若在规定之外，可以通过spi进行定制



### springboot特性

#### 1.springboot的自动装配

> 完成bean的自动装载



#####  条件控制

对于官方组件，是基于**condition**条件来决定对于类是否要自动装配，对于第三方组件，是采用spi机制来实现扩展,通过springfactoriesloader(spring.factories)扫描

1. 官方包spring-boot-starter-xxx

2. 第三方包xxx-spring-boot-starter



##### spi机制

Service provider interface

满足以下条件

1. 需要在classpath目录下创建一个META-INF/services
2. 在该目录下创建一个扩展点的全路径名
   1. 文件中填写这个扩展点的实现
   2. 文件编码格式UTF-8
3. ServiceLoader去进行加载



> spring boot会考虑应用中的其他因素并推断你所需要的Spring配置。比如在A类中有一个成员变量是在jar包中的B类，如果是普通的spring mvc模式，那么你需要在xml中定义B类的一个bean对象，然后才可以在A类中使用@Autowired注解在注入此bean。但是在spring boot默认启动了自动配置，在需要B的时候可以生成B的bean对象并且注入到A中，不需要在xml中做任何配置，如果想要禁用自动配置，就将spring.boot.enableautoconfiguration的值设为false。



#### 2.Starter添加项目依赖

> Starter依赖将所需的常见依赖按组聚集在一起，形成单条依赖



#### 3.Spring Boot CLI与Groovy的高效配合

> Spring Boot CLI充分利用了Spring Boot Starter和自动配置的魔力，并添加了一些Groovy的功能。它简化了Spring的开发流程，通过CLI，我们能够运行一个或多个Groovy脚本，并查看它是如何运行的。在应用的运行过程中，CLI能够自动导入Spring类型并解析依赖。



#### 4.Spring Boot Actuator

> 完成的主要功能就是为基于Spring Boot的应用添加多个有用的管理端点。这些端点包括以下几个内容。

```
maven引入依赖：
groupId: org.springframework.boot
artifactId: spring-boot-actuator

访问url:
GET /autoconfig：描述了Spring Boot在使用自动配置的时候，所做出的决策。
GET /beans：列出运行应用所配置的bean。
GET /configprops：列出应用中能够用来配置bean的所有属性及其当前的值。
GET /dump：列出应用的线程，包括每个线程的栈跟踪信息。
GET /env：列出应用上下文中所有可用的环境和系统属性变量。
GET /env/{name}：展现某个特定环境变量和属性变量的值。
GET /health：展现当前应用的健康状况。
GET /info：展现应用特定的信息。
GET /metrics：列出应用相关的指标，包括请求特定端点的运行次数。
GET /metrics/{name}：展现应用特定指标项的指标状况。
POST /shutdown：强制关闭应用。
GET /trace：列出应用最近请求相关的元数据，包括请求和响应头。
```



