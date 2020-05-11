---
title: Mysql 8 高级-12 触发器
date: 2020-03-19 20:36:26
tags: 
    - Mysql
    - CentOS
typora-root-url: ..
---

### 1. 触发器介绍

​    MySQL的触发器和存储过程一样，都是嵌入到MySQL的一段程序。触发器是由事件来触发某个操作，这些事件包括INSERT、UPDATAE和DELETE语句。如果定义了触发程序，当数据库执行这些语句的时候就会激发触发器执行相应的操作，触发程序是与表有关的命名数据库对象，当表上出现特定事件时，将激活该对象。       

​        触发器（trigger）是一个特殊的存储过程，不同的是，执行存储过程要使用CALL语句来调用，而触发器的执行不需要使用CALL语句来调用，也不需要手工启动，只要当一个预定义的事件发生的时候，就会被MySQL自动调用。比如当对fruits表进行操作（INSERT、DELETE或UPDATE）时就会激活它执行。触发器可以查询其他表，而且可以包含复杂的SQL语句。它们主要用于满足复杂的业务规则或要求。

<!--more-->

### 2. 创建触发器 

创建触发器语法：

```mysql
create trigger trigger_name 

before/after insert/update/delete

on tbl_name 

[ for each row ]  -- 行级触发器

begin

	trigger_stmt ;

end;
```

准备数据：

```mysql
drop table  if exists emp;
create table emp(
  id int(11) not null auto_increment ,
  name varchar(50) not null comment '姓名',
  age int(11) comment '年龄',
  salary int(11) comment '薪水',
  primary key(id)
)engine=innodb default charset=utf8 ;

insert into 
emp(id,name,age,salary) values
(null,'金毛狮王',55,3800),
(null,'白眉鹰王',60,4000),
(null,'青翼蝠王',38,2800),
(null,'紫衫龙王',42,1800);

select * from  emp;
```

示例需求：

```
通过触发器记录 emp 表的数据变更日志 , 包含增加, 修改 , 删除 ;
```

首先创建一张日志表 : 

```sql
create table emp_logs(
  id int(11) not null auto_increment,
  operation varchar(20) not null comment '操作类型, insert/update/delete',
  operate_time datetime not null comment '操作时间',
  operate_id int(11) not null comment '操作表的ID',
  operate_params varchar(500) comment '操作参数',
  primary key(`id`)
)engine=innodb default charset=utf8;
```

创建 insert 型触发器，完成插入数据时的日志记录 : 

```sql
delimiter //

create trigger emp_logs_insert_trigger
    after insert
    on emp
    for each row
begin
    insert into emp_logs (id, operation, operate_time, operate_id, operate_params)
    values (null, 'insert', now(), new.id,
            concat('插入后(id:', new.id, ', name:', new.name, ', age:', new.age, ', salary:', new.salary, ')'));
end //

delimiter ;
```

```mysql
-- 插入数据前 emp_logs表
select * from emp_logs;
-- 插入数据前 emp表
select * from emp;
-- 数据插入emp表
insert into emp(id,name,age,salary) values(null,'光明左使',30,6000);
-- 插入数据后 emp_logs表
select * from emp_logs;
-- 插入数据后 emp表
select * from emp;
```

![image-20200322210338349](/image/mysql/12/120001.png)

创建 update 型触发器，完成更新数据时的日志记录 : 

``` sql
delimiter  //

create trigger emp_logs_update_trigger
    after update
    on emp
    for each row
begin
    insert into emp_logs (id, operation, operate_time, operate_id, operate_params)
    values (null, 'update', now(), new.id,
            concat('修改前(id:', old.id, ', name:', old.name, ', age:', old.age, ', salary:', old.salary, ') , 修改后(id',
                   new.id, 'name:', new.name, ', age:', new.age, ', salary:', new.salary, ')'));
end //

delimiter ;
```

```mysql
-- 更新数据前 emp_logs表
select * from emp_logs;
-- 更新数据前 emp表
select * from emp;
-- 更新emp表
update   emp set id= 5  where  name ='光明左使';
-- 更新数据后 emp_logs表
select * from emp_logs;
-- 更新数据后 emp表
select * from emp;
```

![image-20200322211000648](/image/mysql/12/120002.png)

创建delete 行的触发器 , 完成删除数据时的日志记录 : 

```sql
delimiter //
create trigger emp_logs_delete_trigger
    after delete
    on emp
    for each row
begin
    insert into emp_logs (id, operation, operate_time, operate_id, operate_params)
    values (null, 'delete', now(), old.id,
            concat('删除前(id:', old.id, ', name:', old.name, ', age:', old.age, ', salary:', old.salary, ')'));
end //

delimiter ;
```

```mysql
-- 删除数据前 emp_logs表
select * from emp_logs;
-- 删除数据前 emp表
select * from emp;
-- 删除emp表
delete from emp where id = 5;
-- 删除数据后 emp_logs表
select * from emp_logs;
-- 删除数据后 emp表
select * from emp;
```

![image-20200322211442287](/image/mysql/12/120003.png)



### 3.  查看触发器

​       查看触发器是指查看数据库中已存在的触发器的定义、状态和语法信息等。可以通过命令来查看已经创建的触发器。下面将介绍两种查看触发器的方法，分别是SHOW TRIGGERS和在triggers表中查看触发器信息。

#### 3.1 利用SHOW TRIGGERS语句查看触发器信息

通过SHOW TRIGGERS查看触发器的语句如下：

```mysql
show triggers;
```

```mysql
show triggers\G
```



![image-20200322211730913](/image/mysql/12/120004.png)



#### 3.2  在triggers表中查看触发器信息

 在MySQL中，所有触发器的定义都存在INFORMATION_SCHEMA数据库的TRIGGERS表格中，可以通过查询命令SELECT查看，具体的语法如下：

```mysql
select * from information_schema.triggers where condition;
```

```sql
select * from information_schema.triggers where trigger_name  like 'emp_logs_delete%' \G
```

![image-20200322214800960](/image/mysql/12/120005.png)

​      从上面的执行结果可以得知：TRIGGER_SCHEMA表示触发器所在的数据库；TRIGGER_NAME后面是触发器的名称；EVENT_OBJECT_TABLE表示在哪个数据表上触发；ACTION_STATEMENT表示触发器触发的时候执行的具体操作；ACTION_ORIENTATION是ROW，表示在每条记录上都触发；ACTION_TIMING表示触发的时刻是AFTER；剩下的是和系统相关的信息。

### 4.  删除触发器

使用DROP TRIGGER语句可以删除MySQL中已经定义的触发器，删除触发器语句的基本语法格式如下：

```mysql
drop trigger [schema_name.]trigger_name
```

其中，schema_name表示数据库名称，是可选的。如果省略了schema，将从当前数据库中舍弃触发程序；trigger_name是要删除的触发器的名称。

```mysql
drop trigger staff.emp_logs_delete_trigger;
```



### 666. 彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

