### 一 mysql8.0 解压安装配置

>  注意：以下教程根据自身具体安装目录,自行改动



#### 1 解压安装到/usr/local/dadamn

`mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz`

#### 2 在mysql目录下创建data文件夹

创建data文件夹 存储文件 mkdir data 

#### 3 用户组/用户密码

groupadd mysql

useradd -g mysql mysql

#### 4 授权用户 

chown -R mysql.mysql /usr/local/datamn/mysql



#### 5 初始化基础信息

```
切换到bin目录下 cd bin

./mysqld --user=mysql --basedir=/usr/local/datamn/mysql --datadir=/usr/local/datamn/mysql/data/ --initialize

记住出现的初始密码

```



#### **6 配置mysql**

##### 1. 在mysql/support-files创建文件my-default.cnf

```
[root@rsyncClient support-files]# cd /usr/local/datamn/mysql/support-files/
[root@rsyncClient support-files]# touch my-default.cnf
```

##### 2. 复制配置文件到/etc/my.cnf

```
[root@rsyncClient support-files]# cp -a ./my-default.cnf /etc/my.cnf 
cp: overwrite ‘/etc/my.cnf’? y
```

##### 3. 编辑my.cnf

```
[client]
port=3306
socket=/tmp/mysql.sock

[mysqld]
port=3306
user=mysql
socket=/tmp/mysql.sock
basedir=/usr/local/datamn/mysql
datadir=/usr/local/datamn/mysql/data
```



#### 7 添加mysqld服务到系统 

`cp -a ./support-files/mysql.server /etc/init.d/mysql`



#### 8 授权以及添加服务

`chmod +x /etc/init.d/mysql`

`chkconfig --add mysql`





### 二 启动mysql 

`service mysql start`



#### 1 查看启动状态 

`service mysql status`



#### 2 将mysql命令添加到服务 

`ln -s /usr/local/datamn/mysql/bin/mysql /usr/bin`



#### 3 登录mysql 

`mysql -uroot -p 密码使用之前随机生成的密码`



#### 4 修改root密码 

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456'; `

其中123456是新的密码自己设置

执行 flush privileges; 使密码生效



#### 5 选择mysql数据库 

`use mysql;`



#### 6 修改远程连接并生效 

`update user set host='%' where user='root';`

`flush privileges;`



