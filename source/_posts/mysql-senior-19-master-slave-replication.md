---
title: Mysql 8 高级-19 MySql主从复制
date: 2020-03-31 20:36:26
tags: 
    - Mysql
    - CentOS
typora-root-url: ..
---



### 1. 复制概述

复制是指将主数据库的DDL 和 DML 操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步。

MySQL支持一台主库同时向多台从库进行复制， 从库同时也可以作为其他从服务器的主库，实现链状复制。



### 2. 复制原理

MySQL 的主从复制原理如下。

![1554423698190](/image/mysql/image-20200419232855771.png) 

从上层来看，复制分成三步：

- Master 主库在事务提交时，会把数据变更作为时间 Events 记录在二进制日志文件 Binlog 中。
- 主库推送二进制日志文件 Binlog 中的日志事件到从库的中继日志 Relay Log 。

- slave重做中继日志中的事件，将改变反映它自己的数据。



### 3. 复制优势

MySQL 复制的有点主要包含以下三个方面：

- 主库出现问题，可以快速切换到从库提供服务。

- 可以在从库上执行查询操作，从主库中更新，实现读写分离，降低主库的访问压力。

- 可以在从库中执行备份，以避免备份期间影响主库的服务。



### 4. 搭建步骤

#### 4.1 master

1） 查找配置文件所在位置

```mysql
find  / -name 'my.cnf'
```

![image-20200421194803500](/image/mysql/image-20200421194803500.png)

2)在master 的配置文件（/etc/my.cnf）中，配置如下内容：

```properties
#mysql 服务ID,保证整个集群环境中唯一
server-id=1

#mysql binlog 日志的存储路径和文件名
log-bin=/var/lib/mysql/mysqlbin

#错误日志,默认已经开启
#log-err

#mysql的安装目录
#basedir

#mysql的临时目录
#tmpdir

#mysql的数据存放目录
#datadir

#是否只读,1 代表只读, 0 代表读写
read-only=0

#忽略的数据, 指不需要同步的数据库
binlog-ignore-db=mysql

#指定同步的数据库
#binlog-do-db=db01
```

3） 执行完毕之后，需要重启Mysql：

```sql
service mysqld restart;
```

4） 创建同步数据的账户，并且进行授权操作：

```sql
set global validate_password.policy=0;
set global validate_password.length=1;
create user 'rep'@'192.168.10.79' identified with mysql_native_password by 'wxc@1234';
grant replication slave on *.* to 'rep'@'192.168.10.79';
flush privileges;
```

5） 查看master状态：

```sql
show master status;
```

![image-20200421224317447](/image/mysql/image-20200421224317447.png) 

字段含义：

```
File : 从哪个日志文件开始推送日志文件 
Position ： 从哪个位置开始推送日志
Binlog_Ignore_DB : 指定不需要同步的数据库
```



#### 4.2 slave

1） 在 slave 端配置文件中，配置如下内容：

```properties
#mysql服务端ID,唯一
server-id=2

#指定binlog日志
log-bin=/var/lib/mysql/mysqlbin
```

2）  执行完毕之后，需要重启Mysql：

```mysql
service mysqld restart;
```

3） 执行如下指令 ：

```mysql
change master to master_host= '192.168.10.63', master_user='rep', master_password='wxc@1234', master_log_file='mysqlbin.000003', master_log_pos=847;
```

指定当前从库对应的主库的IP地址，用户名，密码，从哪个日志文件开始的那个位置开始同步推送日志。

4） 开启同步操作

```
start slave;

show slave status;
```

![1554479387365](/H:/资料-MySQL高级教程/文档/assets/1554479387365.png) 

5） 停止同步操作

```
stop slave;
```



#### 4.3 验证同步操作

1） 在主库中创建数据库，创建表，并插入数据 ：

```sql
create database db01;

user db01;

create table user(
	id int(11) not null auto_increment,
	name varchar(50) not null,
	sex varchar(1),
	primary key (id)
)engine=innodb default charset=utf8;

insert into user(id,name,sex) values(null,'Tom','1');
insert into user(id,name,sex) values(null,'Trigger','0');
insert into user(id,name,sex) values(null,'Dawn','1');
```

2） 在从库中查询数据，进行验证 ：

在从库中，可以查看到刚才创建的数据库：

![1554544658640](/H:/资料-MySQL高级教程/文档/assets/1554544658640.png) 

在该数据库中，查询user表中的数据：

![1554544679538](/H:/资料-MySQL高级教程/文档/assets/1554544679538.png) 

