---
title: Mysql 8 高级-11 视图
date: 2020-03-17  20:40:56
tags: 
    - Mysql
    - CentOS

---

### 1.  视图概述

​      视图是从一个或者多个表中导出的，视图的行为与表非常相似，但视图是一个虚拟表。在视图中用户可以使用SELECT语句查询数据，以及使用INSERT、UPDATE和DELETE修改记录。从MySQL 5.0开始可以使用视图，视图可以使用户操作方便，而且可以保障数据库系统的安全。

#### 1.1 视图的含义

​     视图是一个虚拟表，是从数据库中一个或多个表中导出来的表。视图还可以从已经存在的视图的基础上定义。视图一经定义便存储在数据库中，与其相对应的数据并没有像表那样在数据库中再存储一份，通过视图看到的数据只是存放在基本表中的数据。对视图的操作与对表的操作一样，可以对其进行查询、修改和删除。当对通过视图看到的数据进行修改时，相应的基本表的数据也要发生变化；同时，若基本表的数据发生变化，则这种变化也可以自动反映到视图中。

<!--more-->

#### 1.2 视图的作用

​      与直接从数据表中读取相比，视图有以下优点：

1．简单化

​     看到的就是需要的。视图不仅可以简化用户对数据的理解，也可以简化它们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。

2．安全性

​      通过视图用户只能查询和修改他们所能见到的数据。数据库中的其他数据则既看不见也取不到。数据库授权命令可以使每个用户对数据库的检索限制到特定的数据库对象上，但不能授权到数据库特定行和特定的列上。通过视图，用户可以被限制在数据的不同子集上：

（1）使用权限可被限制在基表的行的子集上。

（2）使用权限可被限制在基表的列的子集上。

（3）使用权限可被限制在基表的行和列的子集上。

（4）使用权限可被限制在多个基表的连接所限定的行上。

（5）使用权限可被限制在基表中的数据的统计汇总上。

（6）使用权限可被限制在另一视图的一个子集上，或是一些视图和基表合并后的子集上。

3．逻辑数据独立性

视图可帮助用户屏蔽真实表结构变化带来的影响。

### 2. 创建视图

​     视图中包含了SELECT查询的结果，因此视图的创建基于SELECT语句和已存在的数据表。视图可以建立在一张表上，也可以建立在多张表上。本节主要介绍创建视图的方法。

#### 2.1 创建视图的语法形式

 创建视图使用CREATE VIEW语句，基本语法格式如下： 

```mysql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]

VIEW view_name [(column_list)]

AS select_statement

[WITH [CASCADED | LOCAL] CHECK OPTION]
```

​      其中，CREATE表示创建新的视图；REPLACE表示替换已经创建的视图；ALGORITHM表示视图选择的算法；view_name为视图的名称，column_list为属性列；SELECT_statement表示SELECT语句；WITH[CASCADED | LOCAL] CHECK OPTION参数表示视图在更新时保证在视图的权限范围之内。

​      ALGORITHM的取值有3个，分别是UNDEFINED | MERGE |TEMPTABLE。其中，UNDEFINED表示MySQL将自动选择算法；MERGE表示将使用的视图语句与视图定义合并起来，使得视图定义的某一部分取代语句对应的部分；TEMPTABLE表示将视图的结果存入临时表，然后用临时表来执行语句。

​        CASCADED与LOCAL为可选参数，CASCADED为默认值，表示更新视图时要满足所有相关视图和表的条件；LOCAL表示更新视图时满足该视图本身定义的条件即可。

​      该语句要求具有针对视图的CREATE VIEW权限，以及针对由SELECT语句选择的每一列上的某些权限。对于在SELECT语句中其他地方使用的列，必须具有SELECT权限。如果还有OR REPLACE子句，必须在视图上具有DROP权限。

​     视图属于数据库。在默认情况下，将在当前数据库创建新视图。要想在给定数据库中明确创建视图，创建时应将名称指定为db_name.view_name。

初始化脚本：

```mysql
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
```

```mysql
select  * from  emp;
```

![image-20200322195425370](/image/mysql/image-20200322195425370.png)

创建视图(不显示工资)：

```mysql
create view  emp_view
as
select  name,age from  emp;
```

查询视图:

```mysql
select  * from  emp_view;
```

![image-20200322195848549](/image/mysql/image-20200322195848549.png)



### 3. 查看视图

​     查看视图是查看数据库中已存在的视图的定义。查看视图必须要有SHOWVIEW的权限，MySQL数据库下的user表中保存着这个信息。查看视图的方法包括DESCRIBE、SHOW TABLE STATUS和SHOW CREATE VIEW，本节将介绍查看视图的各种方法。

#### 3.1 使用DESCRIBE语句查看视图基本信息



DESCRIBE可以用来查看视图，具体的语法如下：

```mysql
DESCRIBE 视图名；
```

```mysql
DESCRIBE emp_view;
```

![image-20200322200055034](/image/mysql/image-20200322200055034.png)

​    结果显示出了视图的字段定义、字段的数据类型、是否为空、是否为主/外键、默认值和额外信息。DESCRIBE一般情况下都简写成DESC，输入这个命令的执行结果和输入DESCRIBE的执行结果是一样的。

#### 3.2 使用SHOW TABLE STATUS语句查看视图基本信息



查看视图的信息可以通过SHOW TABLE STATUS的方法完成，具体的语法如下：

```mysql
show table status like '视图名';
```

```mysql
show table status like 'emp_view'\G
```

![image-20200322200308713](/image/mysql/image-20200322200308713.png)

#### 3.3  使用SHOW CREATE VIEW语句查看视图详细信息

使用SHOW CREATE VIEW语句可以查看视图详细定义，语法如下：

```mysql
show create view 视图名;
```

```mysql
show create view emp_view;
```

![image-20200322200508791](/image/mysql/image-20200322200508791.png)

执行结果显示视图的名称、创建视图的语句等信息。



#### 3.4 在views表中查看视图详细信息

​    在MySQL中，information_schema数据库下的views表中存储了所有视图的定义。通过对views表的查询，可以查看数据库中所有视图的详细信息，查询语句如下：

```mysql
select * from   information_schema.views  where  table_name ='视图名';
```

```mysql
select * from   information_schema.views  where  table_name ='emp_view'\G
```

![image-20200322200945761](/image/mysql/image-20200322200945761.png)

### 4.  修改视图

​     修改视图是指修改数据库中存在的视图，当基本表的某些字段发生变化的时候，可以通过修改视图来保持与基本表的一致性。MySQL中通过CREATE OR REPLACE VIEW语句和ALTER语句来修改视图。

#### 4.1  使用CREATE OR REPLACE VIEW语句修改视图

在MySQL中修改视图，可使用CREATE OR REPLACE VIEW语句，语法如下：

```mysql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]

VIEW view_name [(column_list)]

AS select_statement

[WITH [CASCADED | LOCAL] CHECK OPTION]
```

​    可以看到，修改视图的语句和创建视图的语句是完全一样的。当视图已经存在时，修改语句对视图进行修改；当视图不存在时，创建视图。下面通过一个实例来说明。

```mysql
-- 修改前
desc emp_view;
-- 修改视图
create or replace view  emp_view
as
select  name from  emp;
-- 修改后
desc emp_view;
```

![image-20200322201502930](/image/mysql/image-20200322201502930.png)

#### 4.2  使用ALTER语句修改视图

ALTER语句是MySQL提供的另外一种修改视图的方法，语法如下：

```mysql
ALTER [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]

VIEW view_name [(column_list)]

AS select_statement

[WITH [CASCADED | LOCAL] CHECK OPTION]
```

这个语法中的关键字和前面视图的关键字是一样的，这里就不再介绍了

```mysql
-- 修改前
desc emp_view;
-- 修改视图
create or replace view  emp_view
as
select  name,salary from  emp;
-- 修改后
desc emp_view;
```

![image-20200322201700808](/image/mysql/image-20200322201700808.png)

### 5. 更新视图

​         更新视图是指通过视图来插入、更新、删除表中的数据，因为视图是一个虚拟表，其中没有数据。通过视图更新的时候都是转到基本表上进行更新的，如果对视图增加或者删除记录，实际上是对其基本表增加或者删除记录。下面将介绍视图更新的3种方法：INSERT、UPDATE和DELETE。

#### 5.1 UPDATE

```mysql
-- 更新前的表数据
select * from emp;
-- 更新前的视图数据
select * from emp_view;
-- 更新所有人的工资
update emp_view set salary =3000;
-- 更新后的表数据
select * from emp;
-- 更新后的视图数据
select * from emp_view;
```

![image-20200322202406530](/image/mysql/image-20200322202406530.png)

对视图更新后，基本表的内容也更新了，同样当对基本表更新后，另外一个视图中的内容也会更新。

#### 5.2 INSERT 

```mysql
-- 插入前的表数据
select * from emp;
-- 插入前的视图数据
select * from emp_view;
-- 插入一条数据
insert into emp(id,name,age,salary) values(null,'光明左使',30,6000);
-- 插入后的表数据
select * from emp;
-- 插入后的视图数据
select * from emp_view;
```

![image-20200322202839944](/image/mysql/image-20200322202839944.png)

向表中插入一条记录，通过SELECT查看表和视图，可以看到其中的内容也跟着更新.

#### 5.3 DELETE

```mysql
-- 删除前的表数据
select * from emp;
-- 删除前的视图数据
select * from emp_view;
-- 删除一条数据
delete from  emp_view  where name ='光明左使';
-- 删除后的表数据
select * from emp;
-- 删除后的视图数据
select * from emp_view;
```

![image-20200322203301386](/image/mysql/image-20200322203301386.png)

视图中的删除操作最终是通过删除基本表中相关的记录实现的，查看删除操作之后的表和视图，可以看到通过视图删除其所依赖的基本表中的数据。

当视图中包含有如下内容时，视图的更新操作将不能被执行：

（1）视图中不包含基表中被定义为非空的列。

（2）在定义视图的SELECT语句后的字段列表中使用了数学表达式。

（3）在定义视图的SELECT语句后的字段列表中使用聚合函数。

（4）在定义视图的SELECT语句中使用了DISTINCT、UNION、TOP、GROUP BY或HAVING子句。



### 6.  删除视图

当视图不再需要时，可以将其删除。删除一个或多个视图可以使用DROPVIEW语句，语法如下：

```mysql
drop view [if exists]
   view_name[,view_name]...
   [restrict |cascade]
```

其中，view_name是要删除的视图名称，可以添加多个需要删除的视图名称，各个名称之间使用逗号分隔开。删除视图必须拥有DROP权限。



```mysql
-- 删除前查看视图
DESCRIBE emp_view;
-- 删除视图
drop view if exists emp_view;
-- 删除后查看视图
DESCRIBE emp_view;
```

![image-20200322203800358](/image/mysql/image-20200322203800358.png)

可以看到，emp_view视图已经不存在，删除成功。

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

