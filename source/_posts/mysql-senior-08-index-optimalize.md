---
title: Mysql 8 高级-08 索引优化
date: 2020-03-10  20:14:56
tags: 
    - Mysql
    - CentOS
typora-root-url: ..
---

### 1.数据准备

```sql
drop  table if exists person;
create table if not exists person
(
    id    int primary key auto_increment,
    name  varchar(100) not null,
    age   int          not null,
    email varchar(100) not null,
    phone long         not null,
    card  varchar(255) not null,
    memo  varchar(255) default null

) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;
###插入数据
insert into person values (null,'刘一',15,'13888131111@qq.com',13888131111,uuid_short(),null);
insert into person values (null,'陈二',25,'13888132222@qq.com',13888132222,uuid_short(),null);
insert into person values (null,'张三',35,'13888133333@qq.com',13888133333,uuid_short(),null);
insert into person values (null,'李四',45,'13888134444@qq.com',13888134444,uuid_short(),'1111');
insert into person values (null,'王五',55,'13888135555@qq.com',13888135555,uuid_short(),null);
insert into person values (null,'赵六',65,'13888136666@qq.com',13888136666,uuid_short(),null);
insert into person values (null,'孙七',75,'13888137777@qq.com',13888137777,uuid_short(),null);
insert into person values (null,'周八',85,'13888138888@qq.com',13888138888,uuid_short(),null);
insert into person values (null,'吴九',95,'13888139999@qq.com',13888139999,uuid_short(),null);
insert into person values (null,'郑十',99,'13888130000@qq.com',13888130000,uuid_short(),null);

###建立复合索引
create index person__idx_name_age_email
    on person (name, age, email);
###建立唯一索引
create unique index person__uindex_phone
    on person (card);
### 普通索引
create index person_memo_index_memo
    on person (memo);
```

<!--more-->

### 2.全值匹配我最爱

#### 2.1 使用一个字段的索引

```sql
explain select * from person  where name='张三';
```

![image-20200310223728684](/image/mysql/08/080001.png)



#### 2.使用两个字段的索引

```sql
explain select * from person  where name='张三' and age=35;
```

![image-20200310223956838](/image/mysql/08/080002.png)

#### 3.使用三个字段的索引

```sql
explain select * from person  where name='张三' and age=35 and email='13888133333@qq.com';
explain select * from person  where name='张三'and email='13888133333@qq.com' and age=35 ;
explain select * from person  where  age=35 and name='张三'and email='13888133333@qq.com';
```

![image-20200310224239746](/image/mysql/08/080003.png)

注意：虽然有些语句没有按照索引的建立顺序编写查询条件，但是mysql优化器会去优化这个语句。

建议：查询条件的字段顺序最好和建立索引的顺序相同，这样可以避免mysql优化器再次优化。

#### 3.最左前缀要遵守（带头大哥不能死）

​     如果索引了多例，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。

​    以下sql均符合最左前缀法则：

```sql
explain select * from person  where name='张三';
explain select * from person  where name='张三' and age=35;
explain select * from person  where name='张三' and age=35 and email='13888133333@qq.com';
```

![image-20200310225553994](/image/mysql/08/080004.png)

 

将索引最左边的查询字段去掉，使其不符合最左前缀法则，具体如下：

```sql
explain select * from person  where  age=35;
explain select * from person  where  email='13888133333@qq.com';

explain select * from person  where  age=35 and email='13888133333@qq.com';
explain select * from person  where  email='13888133333@qq.com' and age=35 ;

```

![image-20200310225937363](/image/mysql/08/080005.png)

注意：此时不会使用索引，type =ALL，为全表扫描。

#### 4.中间兄弟不能断

```sql
explain select * from person  where name='张三';
explain select * from person  where name='张三' and  email='13888133333@qq.com';
explain select * from person  where name='张三' and age=35;
```

![image-20200310230527686](/image/mysql/08/080006.png)

分析：你会发现第一条语句和第二条语句都只用了name字段做索引，为什么这么说呢？

​           首先第一二条语句都用到了索引，且key_len相等，均等于402，因此说明第二条语句中的email字段没用到索引。再和第三条语句做对比，可以看出使用了2个字段的索引，其key_len为406；

  结论 ：索引中间的字段不能断了，一旦断了，后面的索引将会失效。

#### 5.索引列上少计算

​    索引上尽量避免做函数计算等操作，会导致索引失效而转向全表扫描。

​    不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描。

`WHERE` 条件后面索引列使用函数:

```
explain select * from person  where left(name,1) ='张';
```

![image-20200310231611526](/image/mysql/08/080007.png)

#### 6. 范围之后全失效

不能使用索引中范围条件右边的列，这样会使范围之后的索引全部失效。

```sql
explain select * from person  where name='张三';
explain select * from person  where name='张三' and age=35;
explain select * from person  where name='张三' and age=35 and email='13888133333@qq.com';
explain select * from person  where name='张三' and age>34 and email='13888133333@qq.com';
```

![image-20200310232210678](/image/mysql/08/080008.png)



#### 7. Like百分写最右

查询条件中使用like时要注意，不能写在最左边。

```sql
explain select * from person  where name like '张三';
explain select * from person  where name like '张三%';
explain select * from person  where name like '%张三%';
explain select * from person  where name like '%张三';
```

![image-20200310232929922](/image/mysql/08/080009.png)

#### 8.覆盖索引不写星

尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select*

```sql
explain select name,age,email from person  where name='张三' and age=35 and email='13888133333@qq.com';
explain select * from person  where name='张三' and age=35 and email='13888133333@qq.com';
explain select name,age,email from person  where name='张三' and age>34 and email='13888133333@qq.com';
explain select * from person  where name='张三' and age>34 and email='13888133333@qq.com';
explain select name,age,email from person  where name='张三' and age=35;
explain select * from person  where name='张三' and age=35;
explain select name from person  where name='张三' and age=35;
```

![image-20200310234248964](/image/mysql/08/080010.png)

#### 9.不等空值还有or（分情况讨论）

mysql在使用不等于（！=或者<>）,is null,is not null,or 的时候无法使用索引会导致全表扫描;

##### 9.1 情况一

```sql
select  * from   person;
```

![image-20200311223821049](/image/mysql/08/080011.png)

```sql
explain select name from person  where name='张三' ;
explain select name from person  where name!='张三' ;
explain select name from person  where name<>'张三' ;
explain select  * from   person where memo is not null ;
explain select  * from   person where memo is  null ;
```

![image-20200311223509761](/image/mysql/08/080012.png)

##### 9.2 情况二

修改数据库数据，具体sql如下：

```sql
update person set  memo =uuid_short() where  memo is null;
update person set  memo =null where  memo ='1111';
select  * from   person;
```

![image-20200311224239354](/image/mysql/08/080013.png)

再次执行以下语句，与上面执行结果做对比：

```sql
explain select name from person  where name='张三' ;
explain select name from person  where name!='张三' ;
explain select name from person  where name<>'张三' ;
explain select  * from   person where memo is not null ;
explain select  * from   person where memo is  null ;
```

![image-20200311224433429](/image/mysql/08/080014.png)

通过上述测试可知：

​      在使用 ！= 和<>时并不是不使用索引，而是使用了索引还进行了大量数据的扫描，没有起到明显的索引效果和全表扫描的成本相差不大，或者说没有特别明显的差异。

​       在使用is not null 和 is null  也是一样，如果数据字段对应的值大部门是null ，那么使用is null 查询，此时全表扫描和使用索引的成本差不多，就会出现不走索引的情况；如果大量的数据不是null（not null），那么使用is not null 查询，此时全表扫描和使用索引的成本差不多，就会出现不走索引的情况。

​     or 也是其他2种情况很相似，也是需要考虑全表扫描和索引扫面的成本对比，只能说可能会导致索引失效，并不是一定导致索引失效。



### 10. VAR引号不可丢

不加单引号会隐式转化，导致索引失效；

​        执行一下语句：

```sql
update person set  memo ='1111' where  memo  is null;
select  * from   person;
```

![image-20200311225611738](/image/mysql/08/080015.png)

```sql
explain select  * from   person where memo ='1111' ;
explain select  * from   person where memo =1111 ;
```



![image-20200311225737535](/image/mysql/08/080016.png)



### 11.优化总结口诀

```
全值匹配我最爱，最左前缀要遵守；
带头大哥不能死，中间兄弟不能断；
索引列上少计算，范围之后全失效；
Like百分写最右，覆盖索引不写星；
不等空值还有or，索引失效要少用；
VAR引号不可丢，SQL高级也不难！
```

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```
