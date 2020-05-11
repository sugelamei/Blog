---
title: Mysql 8 高级-09 查询优化
date: 2020-03-12  20:38:56
tags: 
    - Mysql
    - CentOS
typora-root-url: ..
---

### 1.准备数据

```sql
drop table   if exists query;
create table query
(
    id int auto_increment primary key,
    t0 varchar(100),
    t1 varchar(100),
    t2 varchar(100),
    t3 varchar(100),
    t4 varchar(100),
    t5 varchar(100),
    t6 varchar(100),
    t7 varchar(100),
    t8 varchar(100),
    t9 varchar(100)

)engine =innodb default charset =utf8;

insert into query values (null,'a0','a1','a2','a3','a4','a5','a6','a7','a8','a9');
insert into query values (null,'b0','b1','b2','b3','b4','b5','b6','b7','b8','b9');
insert into query values (null,'c0','c1','c2','c3','c4','c5','c6','c7','c8','c9');
insert into query values (null,'d0','d1','d2','d3','d4','d5','d6','d7','d8','d9');
insert into query values (null,'e0','e1','e2','e3','e4','e5','e6','e7','e8','e9');
insert into query values (null,'f0','f1','f2','f3','f4','f5','f6','f7','f8','f9');
insert into query values (null,'g0','g1','g2','g3','g4','g5','g6','g7','g8','g9');
insert into query values (null,'h0','h1','h2','h3','h4','h5','h6','h7','h8','h9');
```

<!--more-->

### 2.创建索引

```sql
create index query_t0_t1_t2_t3_index
	on query (t0, t1, t2, t3);
```

```sql
show  index  from  query;
```

![](/image/mysql/09/090001.png)

### 3.案例分析

#### 3.01 普通案例分析

```sql
explain select  * from query where  t0 ='a0' and t1 ='b1'and t2 ='c2'and t3 ='d3';
explain select  * from query where  t1 ='b1' and t2 ='c2'and t3 ='d3'and t0 ='a0';
explain select  * from query where  t2 ='c2' and t3 ='d3'and t0 ='a0'and t1 ='b1';
explain select  * from query where  t3 ='d3' and t0 ='a0'and t1 ='b1'and t2 ='c2';
```

![image-20200312211651011](/image/mysql/09/090002.png)

分析：创建索引为t0,t1,t2,t3，上述四组explain执行结果都一样，type =ref，key_len=1212,ref=const,const,const,const.

结论：在执行常量等值查询时，改变索引列的顺序并不会更改explain的执行结果，因为mysql底层优化器会进行优化，但是推荐按照索引顺序列编写sql语句。



#### 3.02 范围案例分析

```sql
explain select  * from query where  t0 ='a0';
explain select  * from query where  t0 >'a0' and t1 ='b1'and t2 ='c2'and t3 ='d3';
explain select  * from query where  t0 ='a0' and t1 ='b1';
explain select  * from query where  t0 ='a0' and t1 ='b1'and t2 >'c2'and t3 ='d3';
explain select  * from query where  t0 ='a0' and t1 ='b1'and t2 ='c2';
explain select  * from query where  t0 ='a0' and t1 ='b1'and t2 ='c2'and t3 >'d3';
```

![image-20200312215718145](/image/mysql/09/090003.png)

分析：上图1和2中，1的type =ref,2的为type=range，但是两者的key_len均为303，说明均用了t0索引；

​             上图3和4以及5，3和5的type =ref，4的为type=range，但是4和5 的key_len相同，因此说明4和5都使用了t0,t1,t2索引；

结论：范围右边的索引列失效，但是范围本身是有效的。

#### 3.03 排序案例分析(order by)



```sql
#按照索引顺序排序
explain select  * from query where  t0 >'a0' order by t0;
explain select  * from query where  t0 ='a0' order by t0;
explain select  * from query where  t0 >'a0' order by t0,t1;
explain select  * from query where  t0 ='a0' order by t0,t1;
explain select  * from query where  t0 >'a0' order by t0,t1,t2;
explain select  * from query where  t0 ='a0' order by t0,t1,t2;
explain select  * from query where  t0 >'a0' order by t0,t1,t2,t3;
explain select  * from query where  t0 ='a0' order by t0,t1,t2,t3;
```

![image-20200312222936617](/image/mysql/09/090004.png)



```sql
#未按照索引顺序排序
explain select  * from query where  t0 >'a0' order by t1;
explain select  * from query where  t0 ='a0' order by t1;
explain select  * from query where  t0 >'a0' order by t0,t2;
explain select  * from query where  t0 ='a0' order by t0,t2;
explain select  * from query where  t0 >'a0' order by t2,t0,t1;
explain select  * from query where  t0 ='a0' order by t2,t0,t1;
explain select  * from query where  t0 >'a0' order by t0,t1,t3,t2;
explain select  * from query where  t0 ='a0' order by t0,t1,t3,t2;
```

![image-20200312223343932](/image/mysql/09/090005.png)

```sql
#按照索引排序顺序相反的顺序排序
explain select  * from query where  t0 >'a0' order by t0;
explain select  * from query where  t0 ='a0' order by t0;
explain select  * from query where  t0 >'a0' order by t1,t0;
explain select  * from query where  t0 ='a0' order by t1,t0;
explain select  * from query where  t0 >'a0' order by t2,t1,t0;
explain select  * from query where  t0 ='a0' order by t2,t1,t0;
explain select  * from query where  t0 >'a0' order by t3,t2,t1,t0;
explain select  * from query where  t0 ='a0' order by t3,t2,t1,t0;
```

![image-20200312224004156](/image/mysql/09/090006.png)

```sql
#选择不同的字段排序方式
explain select  * from query where  t0 >'a0' order by t0 desc ;
explain select  * from query where  t0 >'a0' order by t0 asc;
explain select  * from query where  t0 >'a0' order by t0 desc ,t1 desc ;
explain select  * from query where  t0 >'a0' order by t0 asc ,t1 asc;
explain select  * from query where  t0 >'a0' order by t0 asc ,t1 desc ;
explain select  * from query where  t0 >'a0' order by t0 desc ,t1 asc;
```

![image-20200312224727140](/image/mysql/09/090007.png)

分析：综上各个案例可知，当排序的字段未按照索引的建立顺序和索引的排序顺序时会出现Using filesort（文件内排序，会严重影响性能）。当排序字段的顺序与字段索引建立的的排序顺序(默认为ASC)，则会出现 Backward index scan（反向索引扫描）；

总结：

- MySQL支持二种方式的排序，FileSort和Index,Index效率高，它指MySQL扫描索引本身完成排序。FileSort方式效率较低，ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序。
- ORDER BY满足两情况，会使用Index方式排序：ORDER BY语句使用索引最左前列和使用where子句与OrderBy子句条件列组合满足索引最左前列。
- 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

 

> 扩展：
>
> 如果不在索引列上，filesort有两种算法：mysql就要启动双路排序和单路排序
>
> 双路排序：MySQL4.1之前是使用双路排序，字面意思是两次扫描磁盘，最终得到数据。读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据传输，从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。取一批数据，要对磁盘进行两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二张改进的算法，就是单路排序。
>
> 单路排序：从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。
>
> 单路排序出现的问题：当读取数据超过sort_buffer的容量时，就会导致多次读取数据，并创建临时表，最后多路合并，产生多次I/O，反而增加其I/O运算。
>
> 解决方式：
>
> a.增加sort_buffer_size参数的设置。
>
> b.增大max_length_for_sort_data参数的设置。
>
> 提升order by速度的方式：
>
> 1.在使用order by时，不要用select *，只查询所需的字段。因为当查询字段过多时，会导致sort_buffer不够，从而使用多路排序或进行多次I/O操作。
>
> 2.尝试提高sort_buffer_size。
>
> 3.尝试提高max_length_for_sort_data。

#### 3.04排序案例分析(group by)

   group  by实质是先排序后进行分组，遵照索引建的最佳左前缀；

```sql
   explain  select  t0,t1,t2,t3 from query  group by t0,t1,t2,t3 ;
   explain  select  t0,t1,t2 from query  group by t0,t1,t2;
   explain  select  t0,t1 from query  group by t0,t1;
   explain  select  t0 from query  group by t0;
   explain  select  t1,t2,t3 from query  group by t1,t2,t3 ;
   explain  select  t0,t2,t3 from query  group by t0,t2,t3 ;
   explain  select  t0,t1 from query  group by t1,t0;
```

![image-20200315211556400](/image/mysql/09/090008.png)

分析：当使用的分组条件的先后顺序和索引的先后顺序不一样以及未遵循索引的最佳左前缀法则时会出现Using temporary；在实际的生产中要避免这种情况的出现。

> 拓展：
>
> 当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置
>
> where高于having,能写在where限定的条件就不要去having限定。

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

