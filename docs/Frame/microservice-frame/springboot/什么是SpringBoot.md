### 1 什么是SpringBoot

1. 服务于spring框架的框架
2. 约定优于配置下的产物



#### 1.1 如何理解上述两句话

##### 1.1.1 约定优于配置下的产物，怎么理解这句话？

> 如果你想体验springboot带来的轻量级开发，那就必须遵循springboot的自己的约定



##### 1.1.1.1 约定优于配置的体现

1. maven 的目录结构
   a) 默认有 resources 文件夹存放配置文件
   b) 默认打包方式为 jar
2. spring-boot-starter-web 中默认包含 spring mvc 相关依赖以及内置的 tomcat 容器，使得构建一个 web 应用更加简单
3. 默认提供 application.properties/yml 文件
4. 默认通过 spring.profiles.active 属性来决定运行环境时读取的配置文件
5. EnableAutoConfiguration 默认对于依赖的 starter 进行自动装配(不理解先看下文)

   

##### 1.1.2 服务于spring框架的框架，怎么理解这句话？

> 即保留spring的特性，改进spring配置的臃肿，使得文件配置更加轻量化，让我们更专注于业务的开发。



##### 1.1.2.1 springboot是怎么延续spring原有的一些特性呢，并且配置轻量化呢，这就要看到spring的发展历程

1. **Spring 1.x**

   > spring 1.x 注重配置文件，比如说Ioc将一个对象交给容器管理，需要配置文件,以下方式，但是如果项目太大，配置文件就会很臃肿

   ```xml
    <bean id="girl2" class="com.helloworld.User2">
   	<property name="userName" value="Talor Swift"></property>
   </bean>
   ```

   

2. **Spring 2.x**

   > Spring 2.0 配置文件与注解文件并存版本，既可以使用配置文件，也可以使用注解，除了以上方式还多了以下注解

   ```java
   @Controller
   @Service
   @Responstory
   @Compoent
   以上方式省去了配置文件配置，并且可以通过自动扫描来加载对象到Ioc容器
   ```

   

3. **Spring 3.x(里程碑)**

   > Spring 3.x 无配置化的方式实现Bean的装配 ,提供了几个注解以及模块化驱动

   1. @Configuration

      > 它的功能和上述的几个注解功能一样都是通过Ioc来管理对象，它的核心目的就是把bean对象更加便捷的加载到Ioc容器中

   ```java
   @Configuration
   public class RedissonAutoConfiguration {
   
       @Bean
       public RedissonClient redissonClient() {
           return new RedissonClient();
       }
   
   }
   
   以上通过代码的方式来实现Bean(RedissonClient)的装配，通过component-scan扫描注解，加载到Ioc，类似xml，<Bean name="...">配置
   ```

   2. @Import 

      > 导入外部资源包，此资源必须是已经被控制反转

      ```java
      @Import(FormatAutoConfig.class)
      @Configuration
      public class TempFormatProcessor {
      
          @Bean
          public TempDoFormat tempDoFormat(TemplateFormat templateFormat, FormatProperties formatProperties) {
              return new TempDoFormat(templateFormat,formatProperties);
          }
      
      }
      ```

      

   3. @componetScan

      > 定义：默认扫描@Service，@Repository，@Controller，@Bean, @Configuration的注解   添加到IOC

      ```java
      @SpringBootApplication
      @ComponentScan(basePackages = "com.itcrazy.alan.starter")
      public class DemoStarterApplication {
      
          public static void main(String[] args) {
              SpringApplication.run(DemoStarterApplication.class, args);
          }
         
      }
      ```

   4. Enable模块驱动

      1. 怎么理解模块驱动，下面我们来设想一个场景，在spring3.x中需要集成redis如何集成
         1. 创建一个配置类
         2. bean注解声明一个bean

         > 有个问题？ bean注解的方式不能完整的替换第三方包的必要的bean的声明

      2. 所以Enable模块驱动就出现了

         > 把相关组件的bean加载到Ioc容器中,也就是说通过加上@Enablexxxxx注解，比如@EnableRedis就完全将redis相关的组件一起加载到Ioc中，不需要一个一个手动在写配置类了，是不是很便捷

      

4. **Spring 4.x**

   > Spring 3.x中bean的加载，因为没有条件控制，所以不知道在什么样的情况加载，在spring 4x中添加了@conditional 条件注解，控制bean是否要装载

   ```java
   @Configuration
   public class RedissonAutoConfiguration {
   
       // 判断PeCondition(实现Conditional)的match方法是否为true，为true就进行装载redissonClient，否则不装载
       @Conditional(PeCondition.class)
       @Bean
       public RedissonClient redissonClient() {
           return new RedissonClient();
       }
   
   }
   ```

5. **Spring 5.x**

   > 如果需要大扫描配置类，那么@Indexed 可以带来性能提升（不详细写了）



**所以可以看到随着spring的版本迭代，spring想要以最简单的方式来实现bean的构建，那springboot是怎样在spring基础上进行升级的，又利用spring以上版本的哪些特性，来实现配置轻量化呢？看下文springboot的特性** 



### 2 springboot特性

#### 2.1 springboot的自动装配

> 怎么理解，即在启动完成后把需要的包自动加载到Ioc中，即自动装配，和上面我们提到**Enable模块驱动**是不是有点类似，那springboot就在这上面做了延申，即注解**@EnableAutoConfiguration**

##### 2.1.1 **@EnableAutoConfiguration**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
// BeanDefinition 生成所有bean的配置信息
@Import({Registrar.class})
@Import({AutoConfigurationImportSelector.class})

```

@Import 注解是不是很熟悉，上面以及讲过，那小伙伴现在是不是有个疑问就是，springboot是如何知道，扫描的路径的？



##### 2.1.1.1 从何处扫描

AutoConfigurationImportSelector有个方法就实现了，springboot约定存放加载配置类路径的地方，注意这也约定大于配置的一个体现

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

即 在classpath下的META-INF/spring.factories中以key-value的形式存放，通过注解驱动**@EnableAutoConfiguration**的方式扫描，**在springboot中将每个模块都看作一个starter**



##### 2.1.1.2 starter

> starter 分为springboot内置组件和第三方组件，那这两者有什么区别吗

1 命名区别

- 官方包spring-boot-starter-xxx
- 第三方包xxx-spring-boot-starter

2 扫描区别

1. 官方包spring-boot-starter-xxx

   > 官方包中不在spring.factories扫描，集中内置在spring-boot-autoconfiguration中

2. 第三方包xxx-spring-boot-starter

   > 需要创建META-INF/spring.factories 自动加载扫描路径，通过下面的key-value的形式

   `org.springframework.boot.autoconfigure.EnableAutoConfiguration=\com.itcrazy.alan.RedissonAutoConfiguration`

   如果想要使得自己编写的第三方组件，在yml文件能够像我们在使用redis-starter第三方组件，能够进行提示，如下图
   
   ![starter](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/starter.png)
   
   > 1 maven引入spring-boot-configuration-processor 包
>
   > 2 创建additional-spring-configuration-metadata.json
   
   ```json
   {
     "properties": [
       {
         "name": "server.compression.enabled",
         "description": "Whether response compression is enabled.",
         "defaultValue": false
       },
       {
         "name": "server.compression.excluded-user-agents",
         "description": "Comma-separated list of user agents."
       },
       {
         "name": "server.compression.mime-types",
         "description": "Comma-separated list of MIME types that should be compressed.",
         "defaultValue": [
           "text/html",
           "text/xml",
           "text/plain",
           "text/css",
           "text/javascript",
           "application/javascript",
           "application/json",
           "application/xml"
         ]
       }
     ]
}
   ```
   
   

##### 2.1.1.3 控制starter加载

> 上文我们谈到@conditional 条件注解，springboot在此基础上做了延申，来控制starter是否加载，下面我们看一个自定义starter列子如何控制starter的加载

```java
@ConfigurationProperties(prefix = "spring.redisson",ignoreUnknownFields = false)
@Data
public class RedissonProperties {

    private String address; //连接地址

    private int database;

    private int timeout;

    private String password;
}

```

```java
@Configuration
// 使redisson配置生效，使RedissonProperties加载到bean容器中
@EnableConfigurationProperties(RedissonProperties.class)
// 存在reddison包才能加载
@ConditionalOnClass({Redisson.class})
public class RedissonAutoConfiguration {

    @Autowired
    RedissonProperties redissonProperties;

    @Bean
    RedissonClient redissonClient() {
        Config config = new Config();
        String node = redissonProperties.getAddress();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redissonProperties.getTimeout())
                .setConnectionMinimumIdleSize(redissonProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redissonProperties.getPassword())) {
            serverConfig.setPassword(redissonProperties.getPassword());
        }
        return Redisson.create(config);
    }

}
```

**上面的例子可能存在的疑问**

1. @ConfigurationProperties 是干嘛的

   > 如果我们想动态在yml文件配置文件信息，首先要有前缀，再设置需要的参数
   >
   > 可以通过@ConfigurationProperties(prefix = "spring.redisson",ignoreUnknownFields = false)这个注解创建一个属性类提供这样的功能，spring.redisson就是前缀，就可以在yml自定义设置属性值了

2. @EnableConfigurationProperties(RedissonProperties.class)

   > 如果我们想要将属性类引入到配置类，那我们还需要@EnableConfigurationProperties这样的注解，作用就是将RedissonProperties.class加载到Ioc中

3. @ConditionalOnClass({Redisson.class}) 

   > @ConditionalOnClass是不是和@Conditional很像，没错也是在spring上的延申，而ConditionalOnClass就是进行starter控制了，意思是如果你在maven中加了Redisson的依赖，那么就加载，否则不加载



##### 2.1.2 SPI机制

> 我们都知道不同的数据库有不同的驱动，而这些驱动同通常都有第三方来实现，这些不同的驱动可以看作一个扩展点，那我们在面对类似的场景时，springboot是以怎样的方式将这些扩展自动加载呢？

Service provider interface

满足以下条件

1. 需要在classpath目录下创建一个META-INF/services
2. 在该目录下创建一个扩展点的全路径名
   1. 文件中填写这个扩展点的实现
   
   2. 文件编码格式UTF-8
   
      ![spi](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/spi.png)
3. ServiceLoader去进行加载



#### 2.2 Starter添加项目依赖

> Starter依赖将所需的常见依赖按组聚集在一起，形成单条依赖



#### 2.3 Spring Boot CLI与Groovy的高效配合

> Spring Boot CLI充分利用了Spring Boot Starter和自动配置的魔力，并添加了一些Groovy的功能。它简化了Spring的开发流程，通过CLI，我们能够运行一个或多个Groovy脚本，并查看它是如何运行的。在应用的运行过程中，CLI能够自动导入Spring类型并解析依赖。



#### 2.4 Spring Boot Actuator

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



