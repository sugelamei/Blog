---
title: Mysql 8 高级-10 存储过程
date: 2020-03-15  21:24:56
tags: 
    - Mysql
    - CentOS
---

### 1. 存储过程和函数概述

​	存储过程和函数是  事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。	

​	存储过程和函数的区别在于函数必须有返回值，而存储过程没有。

​	函数 ： 是一个有返回值的过程 ；

​	过程 ： 是一个没有返回值的函数 ；

<!--more-->

### 2. 创建存储过程和函数

#### 2.1  创建存储过程

创建存储过程，需要使用CREATE PROCEDURE语句，基本语法格式如下：

```sql
CREATE PROCEDURE sp_name ([proc_parameter[,...]])
begin
	-- SQL语句
end ;
```

CREATE PROCEDURE为用来创建存储函数的关键字；sp_name为存储过程的名称；proc_parameter为指定存储过程的参数列表，列表形式如下：

```mysql
[IN|OUT|INOUT] param_name type
```

其中，IN表示输入参数，OUT表示输出参数，INOUT表示既可以输入也可以输出；param_name表示参数名称；type表示参数的类型，该类型可以是MySQL数据库中的任意类型。

​        编写存储过程并不是一件简单的事情，可能存储过程中需要复杂的SQL语句，并且要有创建存储过程的权限；但是使用存储过程将简化操作，减少冗余的操作步骤，同时，还可以减少操作过程中的失误，提高效率，因此存储过程是非常有用的，而且应该尽可能地学会使用

示例 ：

```sql 
delimiter //

create procedure pro_test1()
begin
	select 'Hello Mysql' ;
end //

delimiter ;
```



<strong><font color="red">知识小贴士</font></strong>

> “DELIMITER //”语句的作用是将MySQL的结束符设置为//，因为MySQL默认的语句结束符号为分号‘;’。为了避免与存储过程中SQL语句结束符相冲突，需要使用DELIMITER改变存储过程的结束符，并以“END//”结束存储过程。存储过程定义完毕之后再使用“DELIMITER ;”恢复默认结束符。DELIMITER也可以指定其他符号作为结束符。当使用DELIMITER命令时，应该避免使用反斜杠（‘\’）字符，因为反斜线是MySQL的转义字符。
>

#### 2.2  创建存储函数 

​    创建存储函数，需要使用CREATE FUNCTION语句，基本语法格式如下：

```mysql
CREATE FUNCTION   func_name([func_parameter])
RETURNS type 
BEGIN
	...
END;
```

CREATE FUNCTION为用来创建存储函数的关键字；func_name表示存储函数的名称；func_parameter为存储过程的参数列表，参数列表形式如下：

```mysql
[IN|OUT|INOUT] param_name type
```

   其中，IN表示输入参数，OUT表示输出参数，INOUT表示既可以输入也可以输出；param_name表示参数名称；type表示参数的类型，该类型可以是MySQL数据库中的任意类型。RETURNS type语句表示函数返回数据的类型。



创建一个存储函数，参数定义为空，返回一个INT类型的结果，获取mysql.user中的数据条数。

```mysql
delimiter //

create function count_user()
returns int
begin
  declare num int ;
  
  select count(*) into num from mysql.user;
  
  return num;
end//

delimiter ;
```

如果在存储函数中的RETURN语句返回一个类型不同于函数的RETURNS子句中指定类型的值，返回值将被强制为恰当的类型。比如，如果一个函数返回一个ENUM或SET值，但是RETURN语句返回一个整数，对于SET成员集相应的ENUM成员，从函数返回的值是字符串。

<strong><font color="red">知识小贴士</font></strong>

> 指定参数为IN、OUT或INOUT只对PROCEDURE是合法的。（FUNCTION中总是默认为IN参数）。RETURNS子句只能对FUNCTION做指定，对函数而言这是强制的。它用来指定函数的返回类型，而且函数体必须包含一个RETURN value语句。

### 3. 调用存储过程和函数

​      存储过程已经定义好了，接下来需要知道如何调用这些过程和函数。存储过程和函数有多种调用方法。存储过程必须使用CALL语句调用，并且存储过程和数据库相关，如果要执行其他数据库中的存储过程，需要指定数据库名称，例如CALL dbname.procname。存储函数的调用与MySQL中预定义的函数的调用方式相同。本节介绍存储过程和存储函数的调用，主要包括调用存储过程的语法、调用存储函数的语法，以及存储过程和存储函数的调用实例。

#### 3.1 调用存储过程

存储过程是通过CALL语句进行调用的，语法如下：

```mysql
call sp_name() ;	
```

CALL语句调用一个先前用CREATE PROCEDURE创建的存储过程，其中sp_name为存储过程名称，parameter为存储过程的参数。

```mysql
-- 调用pro_test1
call pro_test1() ;	
```



![](/image/mysql/image-20200322110532888.png)

#### 3.2 调用存储函数

​      在MySQL中，存储函数的使用方法与MySQL内部函数的使用方法是一样的。换言之，用户自己定义的存储函数与MySQL内部函数是一个性质的。区别在于，存储函数是用户自己定义的，而内部函数是MySQL的开发者定义的。具体语法如下：

```mysql
select   func_name([func_parameter])
```



 **注意**：如果在创建存储函数中报错“you *might* want to use the less safe log_bin_trust_function_creators variable”，需要执行以下代码：

```mysql
set global   log_bin_trust_function_creators=1;
```

调用刚才编写的函数：

```mysql
select  count_user();
```

![image-20200322173236296](/image/mysql/image-20200322173236296.png)

### 4. 查看存储过程和函数

​     MySQL存储了存储过程和函数的状态信息，用户可以使用SHOWSTATUS语句或SHOW CREATE语句来查看，也可直接从系统的information_schema数据库中查询。

#### 4.1  使用SHOW STATUS语句查看存储过程和函数的状态

SHOW STATUS语句可以查看存储过程和函数的状态，其基本语法结构如下：

```mysql
-- 查询存储过程的状态信息
show {procedure|function} status [like 'pattern'];
```

​    这个语句是一个MySQL的扩展，返回子程序的特征，如数据库、名字、类型、创建者及创建和修改日期。如果没有指定样式，那么根据使用的语句，所有存储程序或存储函数的信息都会被列出。其中，PROCEDURE和FUNCTION分别表示查看存储过程和函数；LIKE语句表示匹配存储过程或函数的名称。

```mysql
show procedure status  like 'pro_%'\G
```

![image-20200322165724145](/image/mysql/image-20200322165724145.png)

"show procedure status  like 'pro_%'\G”语句获取数据库中所有名称以字母‘pro_’开头的存储过程的信息。通过上面的语句可以看到：这个存储函数所在的数据库为staaff、存储函数的名称为pro_test1等一些相关信息。

#### 4.2 使用SHOW CREATE语句查看存储过程和函数的定义

​     除了SHOW STATUS之外，MySQL还可以使用SHOW CREATE语句查看存储过程和函数的状态。

```mysql
show create {procedure|function} sp_name
```

​     这个语句是一个MySQL的扩展。类似于SHOW CREATE TABLE，它返回一个可用来重新创建已命名子程序的确切字符串。PROCEDURE和FUNCTION分别表示查看存储过程和函数；sp_name参数表示匹配存储过程或函数的名称。

```mysql
show create function staff.count_user\G
```

![image-20200322173549905](/image/mysql/image-20200322170203698.png)

执行上面的语句可以得到存储函数的名称为count_user，sql_mode为sql的模式，Create Function为存储函数的具体定义语句，还有数据库设置的一些信息。



#### 4.3  从information_schema.Routines表中查看存储过程和函数的信息

​      MySQL中存储过程和函数的信息存储在information_schema数据库下的Routines表中。可以通过查询该表的记录来查询存储过程和函数的信息。其基本语法形式如下：

```mysql
select * from  information_schema.Routines 
where  routine_name='sp_name';
```

其中，ROUTINE_NAME字段中存储的是存储过程和函数的名称；sp_name参数表示存储过程或函数的名称。

```mysql
select * from  information_schema.Routines 
where  routine_name='pro_test1'\G
```

![image-20200322170552335](/image/mysql/image-20200322170552335.png)

​       在information_schema数据库下的Routines表中，存储所有存储过程和函数的定义。使用SELECT语句查询Routines表中的存储过程和函数的定义时，一定要使用ROUTINE_NAME字段指定存储过程或函数的名称。否则，将查询出所有的存储过程或函数的定义。如果有存储过程和存储函数名称相同，就需要同时指定ROUTINE_TYPE字段表明查询的是哪种类型的存储程序。

### 5. 修改存储过程和函数

​    使用ALTER语句可以修改存储过程或函数的特性，下面将介绍如何使用ALTER语句修改存储过程和函数。

```mysql
alter  {procedure|function}  sp_name
```

其中，sp_name参数表示存储过程或函数的名称;

<strong><font color="red">知识小贴士</font></strong>

> 修改存储过程使用ALTER PROCEDURE语句，修改存储函数使用ALTERFUNCTION语句。但是，这两个语句的结构是一样的，语句中的所有参数也是一样的。而且，它们与创建存储过程或函数的语句中的参数也是基本一样的。

### 6. 删除存储过程和函数

删除存储过程和函数，可以使用DROP语句，其语法结构如下：

```mysql
drop {procedure|function}  [if exists] sp_name ;
```

​     这个语句被用来移除一个存储过程或函数。sp_name为要移除的存储过程或函数的名称。IF EXISTS子句是一个MySQL的扩展。如果程序或函数不存储，它可以防止发生错误，产生一个用SHOW WARNINGS查看的警告。

```mysql
drop  procedure if exists pro_test1;
drop  function count_user;
```



### 7. 复杂存储过程语法

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构 ， 来完成比较复杂的功能。

#### 7.1 变量

- DECLARE

  通过 DECLARE 可以定义一个局部变量，该变量的作用范围只能在 BEGIN…END 块中。

```mysql
DECLARE var_name[,...] type [DEFAULT value]
```

示例 : 

```sql
delimiter //
create procedure pro_add()
begin
    declare num int default 5;
    select num + 100;
end //
delimiter ;
```

- SET

直接赋值使用 SET，可以赋常量或者赋表达式，具体语法如下：

```mysql
  SET var_name = expr [, var_name = expr] ...
```

示例 : 

```sql
delimiter //
create procedure pro_set()
begin
    declare name varchar(100);
    set name = 'www.superdevops.cn';
    select name;
end //
delimiter ;
```



也可以通过select ... into 方式进行赋值操作 :

```SQL
delimiter //
create procedure pro_into()
begin
    declare num int;
    select COUNT(*) into num from mysql.user;
    select num;
end //
delimiter ;
```



#### 7.2 if条件判断

语法结构 : 

```mysql
if search_condition then statement_list

	[elseif search_condition then statement_list] ...
	
	[else statement_list]
	
end if;
```

需求： 

```
根据定义的身高变量，判定当前身高的所属的身材类型 

	180 及以上 ----------> 身材高挑

	170 - 180  ---------> 标准身材

	170 以下  ----------> 一般身材
```

示例 : 

```sql
delimiter //
create procedure pro_if()
begin
    declare height int default 175;
    declare descrip varchar(50);
    if height >= 180 then
        set descrip = '身材高挑';
    elseif height >= 170 and height < 180 then
        set descrip = '标准身材';
    else
        set descrip = '一般身材';
    end if;
    select descrip;
end //
delimiter ;
```

调用结果为 : 

 ![image-20200322180358688](/image/mysql/image-20200322180358688.png)



#### 7.3 传递参数

语法格式 : 

```
create procedure procedure_name([in/out/inout] 参数名   参数类型)
...


IN :   该参数可以作为输入，也就是需要调用方传入值 , 默认
OUT:   该参数作为输出，也就是该参数可以作为返回值
INOUT: 既可以作为输入参数，也可以作为输出参数
```

**IN - 输入**

需求 :

```
根据定义的身高变量，判定当前身高的所属的身材类型 
```

示例  : 

```sql
delimiter //
create procedure pro_if_in(in height int)
begin
    declare descrip varchar(50) default '';
    if height >= 180 then
        set descrip = '身材高挑';
    elseif height >= 170 and height < 180 then
        set descrip = '标准身材';
    else
        set descrip = '一般身材';
    end if;
    select concat('身高: ', height, '对应的身材类型为: ', descrip);
end //
delimiter ;
```

调用示例：

```mysql
call pro_if_in(159);
```

![image-20200322181006721](/image/mysql/image-20200322181006721.png)

**OUT-输出**

 需求 :

```
根据传入的身高变量，获取当前身高的所属的身材类型  
```

示例:

```SQL 
delimiter //
create procedure pro_if_out(in height int, out descrip varchar(50))
begin
    if height >= 180 then
        set descrip = '身材高挑';
    elseif height >= 170 and height < 180 then
        set descrip = '标准身材';
    else
        set descrip = '一般身材';
    end if;
end //
delimiter ;
```

调用:

```
call pro_if_out(168, @descrip);
select @descrip;
```

<font color='red'>**小知识** </font>

> @descrip :  这种变量要在变量名称前面加上“@”符号，叫做用户会话变量，代表整个会话过程他都是有作用的，这个类似于全局变量一样。
>
> @@global.sort_buffer_size : 这种在变量前加上 "@@" 符号, 叫做 系统变量 
>



#### 7.4 case结构

语法结构 : 

```mysql
方式一 : 

CASE case_value

  WHEN when_value THEN statement_list
  
  [WHEN when_value THEN statement_list] ...
  
  [ELSE statement_list]
  
END CASE;


方式二 : 

CASE

  WHEN search_condition THEN statement_list
  
  [WHEN search_condition THEN statement_list] ...
  
  [ELSE statement_list]
  
END CASE;

```

需求:

```
给定一个月份, 然后计算出所在的季度
```

示例  :

```sql
delimiter //
create procedure pro_case(in month int)
begin
    declare result varchar(50);
    case
        when month >= 1 and month <= 3 then
            set result = '第一季度';
        when month >= 4 and month <= 6 then
            set result = '第二季度';
        when month >= 7 and month <= 9 then
            set result = '第三季度';
        when month >= 10 and month <= 12 then
            set result = '第四季度';
        when month > 12 then
            set result = '不存在';
        end case;
    select concat('您输入的月份为:  ', month, '  该月份为:  ', result) as content;

end //
delimiter ;
```



#### 7.5 while循环

语法结构: 

```sql
while search_condition do

	statement_list
	
end while;
```

需求:

```
计算从1加到n的值
```

示例  : 

```sql
delimiter  //
create procedure pro_while(in n int)
begin
    declare total int default 0;
    declare num int default 1;
    while num <= n
        do
            set total = total + num;
            set num = num + 1;
        end while;
    select total;
end //
delimiter  ;
```



#### 7.6 repeat结构

有条件的循环控制语句, 当满足条件的时候退出循环 。while 是满足条件才执行，repeat 是满足条件就退出循环。

语法结构 : 

```mysql
REPEAT

  statement_list

  UNTIL search_condition

END REPEAT;
```

需求: 

```
计算从1加到n的值
```

示例  : 

```sql
delimiter //
create procedure pro_repeat(in n int)
begin
    declare total int default 0;
    repeat
        set total = total + n;
        set n = n - 1;

    until n = 0 end repeat;
    select total;
end //
delimiter  ;
```



#### 7.7 loop语句

LOOP 实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如下：

```mysql
[begin_label:] LOOP

  statement_list

END LOOP [end_label]
```

如果不在 statement_list 中增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。



#### 7.8 leave语句

用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简单例子 , 退出循环：

```SQL
delimiter //
create procedure pro_loop_leave(in n int)
begin
    declare total int default 0;
    ins:
    loop
        if n <= 0 then
            leave ins;
        end if;
        set total = total + n;
        set n = n - 1;

    end loop ins;
    select total;
end //

delimiter ;
```



#### 7.9 游标/光标

游标是用来存储查询结果集的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用包括光标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

声明光标：

```mysql
DECLARE cursor_name CURSOR FOR select_statement ;
```

OPEN 光标：

```mysql
OPEN cursor_name ;
```

FETCH 光标：

```mysql
FETCH cursor_name INTO var_name [, var_name] ...
```

CLOSE 光标：

```mysql
CLOSE cursor_name ;
```

 

初始化脚本:

``` sql
create table emp(
  id int(11) not null auto_increment ,
  name varchar(50) not null comment '姓名',
  age int(11) comment '年龄',
  salary int(11) comment '薪水',
  primary key(`id`)
)engine=innodb default charset=utf8 ;

insert into 
emp(id,name,age,salary) values
(null,'金毛狮王',55,3800),
(null,'白眉鹰王',60,4000),
(null,'青翼蝠王',38,2800),
(null,'紫衫龙王',42,1800);

```

通过循环结构 , 获取游标中的数据 : 

```sql
-- 查询emp表中数据, 并逐行获取进行展示
delimiter  //
create procedure pro_emp()
begin
    declare e_id int(11);
    declare e_name varchar(50);
    declare e_age int(11);
    declare e_salary int(11);
    declare num int;
    declare emp_result cursor for select * from emp;
    select count(*) into num from emp;
    open emp_result;
    while num > 0
        do

            fetch emp_result into e_id,e_name,e_age,e_salary;
            select concat('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为: ', e_salary);
            set num = num - 1;
        end while;

    close emp_result;
end //
```

调用后结果如下：

![image-20200322193425687](/image/mysql/image-20200322193425687.png)

注意：这个语句在客户端运行会报错。

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

