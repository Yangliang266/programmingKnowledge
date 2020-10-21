#### 新增

```mysql
INSERT INTO student1 (name) VALUES('tom');
INSERT INTO B (name,code) SELECT name,codeFROM A;
```



#### 删除

```mysql
delete from A where id=2;
TRUNCATE TABLE student1;
drop table if exists practise1;
```



#### 修改

```mysql
UPDATE student1 set name='tom' WHERE id=1;
```



#### 查询



##### 简单查询

1. 全局查询

   ```mysql
   SELECT * FROM student2;
   ```

   

2. 避免重复

   ```mysql
   SELECT DISTINCT age FROM student2;
   ```

   

3. 指定字段

   ```mysql
   SELECT * FROM student2 where id = 1;
   ```

   



##### 条件查询

1. 关系逻辑运算符

   ```mysql
   SELECT id FROM student2 WHERE NOT(id >= 5 );
   ```

   

2. between and

   ```mysql
   SELECT id,name,code FROM student2 WHERE id BETWEEN 6 AND 20;
   ```

   

3. 关键字查询

   ```mysql
   SELECT id,name FROM student2 WHERE id in(1,2,3,4);
   ```

   

4. like查询

   ```mysql
   SELECT id,name FROM student2 WHERE name LIKE '%张%';
   ```

   

5. is null

   ```mysql
   select id,name,code from practise1 where tel is not null;
   ```

   

##### 排序查询

```mysql
select id,name,code,num from practise1 order by id desc,num desc;
```

>可以多个关键字排序，若第一个数据相同，第二个进行排序

##### 限制查询

```mysql
select id,name,code from practise1 limit 10,5;
```

> 从第10条开始查询，查询5行

##### 统计（聚合函数）

```mysql
select avg(salary) as 'avg', max(salary) as 'max', min(salary) as 'min',count(id) as '记录人数' from practise1 where num=1;
```

> 有效行数

##### 分组查询

```mysql
select num,sex,avg(salary) as 'avg',  min(salary) as 'min' from practise1 where salary > 100 group by num,sex having avg(salary) > 200;
```

> 分组查询：分组列带聚合函数
>
> 分组前条件用where分组后条件用having 带聚合函数



##### 外连接查询

1. 种类

```mysql
select * from t_employee as e inner join t_depart as d on d.id = e.dep_id;
select * from t_employee as e left join t_depart as d on d.id = e.dep_id;
select * from t_employee as e right join t_depart as d on d.id = e.dep_id;
```



2. 区别

   left以左表为主表，从左表返回所有的行

   right以右表为主表，从右表返回所有的行

   inner 返回两个表都存在数据的列信息

   

3. 注意点

   from..left join..，left前为左表即主表.



##### 子查询

1. 种类

```mysql
select d.name  from t_depart as d where d.id=(select e.dep_id from t_employee as e where e.name = 'xx');
select e.name from t_employee as e where e.dep_id in (select d.id from t_depart as d where d.name = 'a' or d.name = 'b');
select d.name as '部门' from t_depart as d where exists (select * from t_employee as e where d.id = e.dep_id);
```



2. 区别

   第一种为标量查询

   第二种为列子查询

   第三种为存在查询

3. 注意点

   单表执行，嵌套,(not) in,（not）exists

   

   单行单列

   多行单列

   多在where 后使用子查询

   

   多行多列

   多在from 后使用子查询



##### 合并查询

1. 种类

   union

   union all

2. 区别

   使用union关键字是，会将所有的查询结果合并到一起（相同类型结果集的合并），然后去掉相同的记录；

   使用union all，不会去除掉重复的记录；

3. 注意点

   两表之间的列必须对应



##### 表关系

一对一  外键既是主键又是外键（分为主从表）

多对多  需要有中间表记录各主表之间的外键进行关联