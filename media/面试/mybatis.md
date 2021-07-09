### 1 什么是 Mybatis？  

对象关系映射框架

对象指的是程序中的对象，关系是对象和数据库之间的关系，即对象和和关系型数据库之间的映射



### 2 Mybaits 的优点  

#### 1 使用层面

数据库的连接，预编译，语句集，结果集都封装在一起，可以更专注于sql的编写

#### 2 性能方面

##### 2.1  预编译

合并多次操作为一次操作，dbms不需要再次编译

对于使用同一个sql，缓存起来，preparestate

##### 2.2 动态sql

在预编译之前

sql动态解析

参数检验

占位符处理

动态sql的处理



### 3 MyBatis 框架的缺点  



### 4 MyBatis 框架适用场合  

业务场景复杂，需要大量编写复杂的sql语句





### 5 \#{}和${}的区别是什么？  

`#{}`预编译处理，sql的动态解析，会在sql中加单引号，会将#{} 的值解析为 ？

`${}`字符串替换，sql的动态解析，作为sql本身不会加单引号

变量的替换会将${} 的值解析为 括号内具体的值，容易出现sql注入，比如出现`select * from test where name = 'test', delete test` ,括号内的值可能为{test,delete test}



### 6 通常一个Xml 映射文件，都会写一个Dao 接口与之对应，请问，这个Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗  

Dao对应的是Mapper文件，Mapper是发送sql操作数据库的，可以看作是一个代理对象，Mapper文件中的namespace的看作是接口名，statementid看作是接口的方法名，方法内的参数是传递给sql的参数

mapper接口是没有实现的，当调用接口方法是，接口名+方法名拼接生成key，通过key生成唯一的mapperstatement，select，delete，update，insert都会被解析成一个mapperstatement对象

mapper接口的方法不能重载，因为mybatis为mapper生成一个jdk动态代理对象proxy，代理对象会拦截接口方法，执行mapperstatement所代表的sql，将sql执行结果返回



### 7 Mybatis 是如何将sql 执行结果封装为目标对象并返回的？都有哪些映射形式？  
1 resultMap标签

逐一定义对象名和数据库列名之间的映射关系，属性映射

2 resultType

通过查询结果集中每条记录（属性）的数据类型和Bean对象的数据类型作映射，要求Bean对象字段名和查询结果集的属性名相同

3 别名，将列的别名定义为对象名





### 8 在 mapper 中如何传递多个参数  

1 在mapper接口文件，使用@param 注解，一个参数对应一个@param 注解

2 将多个参数放入到一个dto对象中，通过parameterType获取



### 9 Mybatis 动态sql 有什么用？执行原理？有哪些动态sql？  

可以在xml文件中以标签的形式编写sql，根据表达式的值完成逻辑判断动态拼接sql

trim，set，foreach，if，when，where，otherwise，choose，bind



### 10 Mybatis 的 Xml 映射文件中， 不同的 Xml 映射文件， id 是否可以重复？  

1 在同一个namespace中，id不可以重复，因为接口名+id生成的key，是构成mapperstatement的唯一标识，如果重复会被覆盖

2 在不同的namespace中，id可以重复，namespace不同，id相同构成的标识也不同，所以可以重复



### 11 为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？  
mybatis在查询关联对象或者关联集合时，sql往往需要自己手动编写，所以称为半自动orm映射工具

Hibernate 属于全自动 ORM 映射工具， 使用 Hibernate 查询 关联 对象或 者关 联集合对象时， 可以根据对象关系模型直接获取， 所以它是全自动的  



### 12 一对一、一对多的关联查询 ？  

association标签， collection标签



### 13 MyBatis 实现一对一有几种方式?具体怎么操作的？  

联合查询和嵌套查询

联合查询只查询一次，通过resultMap的association标签配置一对一的类即可

嵌套查询，先查询一个表，将查到的外键作为条件，再到另一个表中查询，也是通过association配置，另一个表查询是以select属性配置



### 14 MyBatis 实现一对多有几种方式,怎么操作的？  

联合查询只查询一次，通过resultMap的collection标签配置一对多的类即可

嵌套查询，先查询一个表，将查到的外键作为条件，再到另一个表中查询，也是通过collection配置，另一个表查询是以select属性配置



### 15 Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？  
Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一， collection 指的就是一对多查询。在 Mybatis 配置文件中， 可以配置是否启用延迟加载 lazyLoadingEnabled=true|false  

**延迟加载**：在真正使用数据的时候才发起查询，不用的时候不查询关联的数据，延迟加载又叫按需查询（懒加载）

**使用场景**：在对应的四种表关系中，一对多、多对多通常情况下采用延迟加载，多对一、一对一通常情况下采用立即加载。



**原理：**使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用A.getB().getName()，拦截器invoke()方法发现A.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用A.setB(b)，于是a的对象b属性就有值了，接着完成A.getB().getName()方法的调用。这就是延迟加载的基本原理。



### 16 Mybatis 的一级、二级缓存  

一级缓存作用于sqlsession，对于相同的sqlsession，执行相同的查询SQL，第一次会去查询数据库，并写到缓存中；第二次直接从缓存中取。当执行SQL时两次查询中间发生了增删改操作，则SqlSession的缓存清空

Mybatis的二级缓存是指mapper映射文件。二级缓存的作用域是同一个namespace下的mapper映射文件内容，多个SqlSession共享。Mybatis需要手动设置启动二级缓存。在SQL 映射xml文件中添加以下一行<cache/>

对于缓存数据更新机制， 当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear 。  



### 17 什么是 MyBatis 的接口绑定？有哪些实现方式？  

在Mapper文件中定义接口，将接口方法和sql绑定起来，直接调用接口实现

注解绑定 @select等等

xml的sql绑定，xml的namespace必须为接口的全路径名



### 18 使用 MyBatis 的mapper 接口调用时有哪些要求？  

1 接口的方法名和mapperstatement的id相同

2 接口的输入参数类型必须和parameterType的类型相同

3 接口的输出参数类型必须和resultType类型相同

4 mapper接口的类路径和namespace的全路径名相同



### 19 Mapper 编写有哪几种方式？  

使用 mapper 扫描器  

1、mapper.xml 文件编写：
mapper.xml 中的 namespace 为 mapper 接口的地址；
mapper 接口中的方法名和 mapper.xml 中的定义的 statement 的 id 保持一致； 如果将mapper.xml 和 mapper 接口的名称保持一致则不用在 sqlMapConfig.xml 中进行配置。
2、定义 mapper 接口：
注意 mapper.xml 的文件名和 mapper 的接口名称保持一致， 且放在同一个目录

3、配置 mapper 扫描器：  

```xml
<bean id="sqlSessionFactory_card" class="org.mybatis.spring.SqlSessionFactoryBean" lazy-init="true">
    <property name="dataSource" ref="dynamicDataSource" />
    <property name="mapperLocations">
        <list>
            <value>classpath*:/com/meishi/card/sqlmap*.xml</value>
            <value>classpath*:/com/meishi/cardReport/sqlmap*.xml</value>
        </list>
    </property>
</bean>
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactory" ref="sqlSessionFactory_card" />
    <property name="basePackage" value="com.meishi.card.dao,com.meishi.cardReport.dao" />
</bean>
```


### 20 简述 Mybatis 的插件运行原理，以及如何编写一个插件  



### 21 