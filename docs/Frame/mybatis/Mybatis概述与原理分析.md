## Mybatis



### 1 什么是 mybatis

#### 概述

> ORM框架，对象与关系的映射，对象是程序中的对象，关系是它与数据库之间的关系，即程序对象的与关系型数据库之间的映射关系

#### 功能架构

![图片描述](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/55b1dbbd00016b6407090390.png)



### 为什么使用 mybatis

#### 有什么特点

1. 高度封装（JDBC）

   > 重复代码，资源管理，结果集处理，sql耦合



#### 好用的原因

##### 底层的原理是什么

1. 底层原理流程

   ![img](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/mybatis/u=823500757,1778223719&fm=26&fmt=auto&gp=0.jpg)

1. SqlSessionFactorybuilder

   > 用来构建SqlSessionFactory， 存在于方法的局部

2. SqlSessionFactory（单例）

   > 用来构建SqlSession， 每次应用程序访问数据库，都需要创建一个会话，作用域是应用作用域

4. SqlSession（粗粒度）

   > 表示一个会话，线程不安全，不能线程共享，在请求开始创建，结束后关闭。一次请求或者操作中。（可以表示为一个select或者insert，update语句）
   >
   > SqlSession 的作用类似于一个 JDBC 中的 Connection 对象，代表着一个连接资源的启用。具体而言，它的作用有 3 个：
   >
   > - 获取 Mapper 接口。
   > - 发送 SQL 给数据库。
   > - 控制数据库事务。
   >
   > 两种发送 SQL 的方式，一种用 SqlSession 直接发送，
   >
   > 另外一种通过 SqlSession 获取 Mapper 接口再发送。

4. Mapper（代理对象，细粒度）

   > 发送sql操作数据库数据，sqlsession事务方法之内



### 2 如何使用好 mybatis



#### 2.1 配置方面

1. type-aliases-package 别名

   ```xml
   type-aliases-package: com.itcrazy.alanmall.order.dal.entity
   resultMap中的type 或者 statement中的parameterType 都可以被替换，比如
   <insert id="mqInsert" parameterType="MqMessage"> //com.itcrazy.alanmall.order.dal.entity中寻找MqMessage
   ```

2. 自定义类型转换规则

   实现BaseTypeHandler

   插入值时加入typeHandler标签，添加路径

   返回值时在result 标签添加typeHandler标签
   
   

#### 2.2 传参方面

> #{}和${}

##### 2.2.1 动态SQL

> 动态 SQL 是 mybatis 的强大特性之一，也是它优于其他 ORM 框架的一个重要原因。mybatis 在对 sql 语句进行预编译之前，会对 sql 进行动态解析，解析为一个 BoundSql 对象，也是在此处对动态 SQL 进行处理的。

###### 1 动态sql 的动态解析

mybatis 在调用 connection 进行 sql 预编译之前，会对sql语句进行动态解析，动态解析主要包含如下的功能：

- 占位符的处理
- 动态sql的处理
- 参数类型校验



###### 2 动态sql解析阶段的不同表现

在MyBatis 的映射配置文件中，动态传递参数有两种方式：

（1）#{} 占位符  

（2）${} 拼接符  

在动态 SQL 解析阶段， #{ } 和 ${ } 会有不同的表现：

> **#{ } 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符。**

例如，sqlMap 中如下的 sql 语句

```sql
select * from user where name = #{name};
```

解析为：

```sql
select * from user where name = ?;
```

一个 #{ } 被解析为一个参数占位符 `?` 。

而，

> **${ } 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换**

例如，sqlMap 中如下的 sql

```sql
select * from user where name = ${name};
```

当我们传递的参数为 "ruhua" 时，上述 sql 的解析为：

```sql
select * from user where name = "ruhua";
```

预编译之前的 SQL 语句已经不包含变量 name 了。



  

  ###### 3 #{} 和 ${} 的区别

>**#{}** 就是编译好SQL语句再取值.
>**${}** 就是取值以后再去编译SQL语句.



> #{} 为参数占位符 ?，即sql 预编译  
>
> ${} 为字符串替换，即 sql 拼接  

  

> #{}：动态解析 -> 预编译 -> 执行  
>
> ${}：动态解析 -> 编译 -> 执行  

   

> #{} 的变量替换是在DBMS 中  
>
> ${} 的变量替换是在 DBMS 外  

  

> 变量替换后，#{} 对应的变量自动加上单引号 ''
>
> 变量替换后，${} 对应的变量不会加上单引号 ''

  

> #{} 能防止sql 注入
>
> ${} 不能防止sql 注入



###### 4 #{} 和 ${} 在使用中的技巧和建议

1 不论是单个参数，还是多个参数，一律都建议使用注解@Param("")

2 能用 `#{}` 的地方就用 `#{}`，不用或少用 `${}`

3 表名作参数时，必须用`${}` ${}。如：select * from ${tableName}

4 order by 时，必须用 `${}`。如：select * from t_user order by ${columnName}

5 使用 ${} 时，要注意何时加或不加单引号，即 ${} 和 '${}'



##### 2.2.2 sql预编译

###### 定义

> sql 预编译指的是数据库驱动在发送 sql 语句和参数给 DBMS 之前对 sql 语句进行编译，这样 DBMS 执行 sql 时，就不需要重新编译。

###### 为什么需要预编译

JDBC 中使用对象 PreparedStatement 来抽象预编译语句，使用预编译

1. **预编译阶段可以优化 sql 的执行**。
   预编译之后的 sql 多数情况下可以直接执行，DBMS 不需要再次编译，越复杂的sql，编译的复杂度将越大，预编译阶段可以合并多次操作为一个操作。
2. **预编译语句对象可以重复利用**。
   把一个 sql 预编译后产生的 PreparedStatement 对象缓存下来，下次对于同一个sql，可以直接使用这个缓存的 PreparedState 对象。

> mybatis 默认情况下，将对所有的 sql 进行预编译。



#### 2.3 性能方面

1. **批量新增**

   ```xml
   <insert id="addPersons">
     insert into person(username,email,gender) VALUES
      <foreach collection="persons" item="person" separator=";">
        (#{person.username},#{person.email},#{person.gender})
       </foreach>
    </insert>
   ```

   

2. **批量更新**

   **2.1 case when then end**

   ```xml
   <update id="updateBatch">
       update t_calendar_extend
       <trim prefix="set" suffixOverrides=",">
           <trim prefix="modify_time = case index" suffix="end,">
               <foreach collection="list" item="item">
                   when #{item.index} then #{item.modifyTime}
               </foreach>
           </trim>
           <trim prefix="user_type = case index" suffix="end">
               <foreach collection="list" item="item">
                   when #{item.index} then #{item.type}
               </foreach>
           </trim>
       </trim>
       <where>
           index in (
           <foreach collection="list" separator="," item="item">
               #{item.index}
           </foreach>
           )
       </where>
   </update>
   
   update t_calendar_extend
   set
   modify_time =
   case index
   when ? then ?
   when ? then ?
   when ? then ? end,
   user_type =
   case index
   when ? then ?
   when ? then ?
   when ? then ? end
   where index in (?,?,?)
   
   
   
   
   update tb_class
   set
   modify_time =
   case id
   when ? then ? 
   when ? then ?
   when ? then ? end,
   user_type =
   case id
   when ? then ?
   when ? then ?
   when ? then ? end
   where id in (?,?,?)
   ```

   > 这种方式实现的批量更新操作效率很低，而且，当更新的字段很多时，SQL语句会特别长。

   > 这里，根据index值来更新modify_time 和user_type的值，有几个字段，就要遍历几遍，效率很低。

   

   **2.2  foreach成多条sql**

   ```xml
   <update id="updateBatch"  parameterType="java.util.List">  
       <foreach collection="list" item="item" index="index" open="" close="" separator=";">
           update tableName
           <set>
               name=${item.name},
               name2=${item.name2}
           </set>
           where id = ${item.id}
       </foreach>      
   </update>
   ```

   > 这种方式最简单，就是用foreach组装成多条update语句，但Mybatis映射文件中的sql语句默认是不支持以" ; " 结尾的，也就是不支持多条sql语句的执行。所以需要在连接mysql的url上加 &allowMultiQueries=true 这个才可以执行。

   

   **2.3 ON DUPLICATE KEY UPDATE**

   ```xml
   <update id="updateBatch">
       insert into t_output_calendar (index, 
         cal_date, user_type, create_time, 
         modify_time, delete_flag
         )
       values
       <foreach collection="list" item="item" index="index"
           separator=",">
           (
           #{item.index,jdbcType=INTEGER}, 
           #{item.calDate,jdbcType=TIMESTAMP}, 
           #{item.type,jdbcType=TINYINT}, 
           #{item.createTime,jdbcType=TIMESTAMP}, 
           #{item.modifyTime,jdbcType=TIMESTAMP}, 
           #{item.deleteFlag,jdbcType=TINYINT}
           )
       </foreach>
       ON DUPLICATE KEY UPDATE modify_time = VALUES(modify_time), user_type = VALUES(user_type);
   </update>
   ```

   MYSQL中的ON DUPLICATE KEY UPDATE，是基于主键（PRIMARY KEY）或唯一索引（UNIQUE INDEX）使用的。

   **如果已存在该唯一标示或主键就更新，如果不存在该唯一标示或主键则作为新行插入。**

3. **批量删除**

   ```xml
   <delete id="deleteMoreEmp" parameterType="int[]">
   		<!-- delete from emp where empno in(7789,7790) -->
   		<!-- forEach : 用来循环 collection : 用来指定循环的数据的类型 可以填的值有：array,list,map item 
   			: 循环中为每个循环的数据指定一个别名 index : 循环中循环的下标 open : 开始 close : 结束 separator : 数组中元素之间的分隔符 -->
       delete from emp where empno in
       <foreach collection="array" item="arr" index="no" open="(" separator="," close=")">
           #{arr}
       </foreach>
   </delete>
   
   ```

   

#### 2.4 其他标签使用

```xml
DATE_FORMAT(rrh.CREATE_TIME,"%Y-%m-%d") <![CDATA[ <= ]]> cast(#{timeEnd} as date)
```

1. DATE_FORMAT 日期格式转换
2. <![CDATA[ <= ]]> 转义符 把 [] 内容原样输出， 比进行解析
3. cast  转换类型









