---
title: Mysql高级-06 性能优化
date: 2020-03-03  20:14:56
tags: 
    - Mysql
    - CentOS
---

### 1. `MySQL` Query Optimizer

​      MySQL Optimizer是一个专门负责优化SELECT 语句的优化器模块，它主要的功能就是通过计算分析系统中收集的各种统计信息，为客户端请求的Query 给出他认为最优的执行计划，也就是他认为最优的数据检索方式。

​      当客户端向MySQL 请求一条Query，命令解析器模块完成请求分类，区别出是 SELECT 并转发给MySQL Query Optimizer时，MySQL Query Optimizer 首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对 Query 中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析 Query 中的 Hint 信息（如果有），看显示Hint信息是否可以完全确定该Query 的执行计划。如果没有 Hint 或Hint 信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据 Query 进行写相应的计算分析，然后再得出最后的执行计划。Query Optimizer 是一个数据库软件非常核心的功能，虽然说起来只是简单的几句话，但在 MySQL 内部，MySQL Query Optimizer 实际上经过了很多复杂的运算分析，才得出最后的执行计划。

<!--more-->

### 2.`MySQL`常见瓶颈



​     CPU:CPU在饱和的时候一般发生在数据装入在内存或从磁盘上读取数据时候

​    IO:磁盘I/O瓶颈发生在装入数据远大于内存容量时

​    服务器硬件的性能瓶颈：top,free,iostat和vmstat来查看系统的性能状态

### 3. Explain

#### 3.1 Explain介绍

​      使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是
如何处理你的SQL语句的，分析你的查询语句或是结构的性能瓶颈。

#### 3.2 Explain的作用

通过explain+sql语句可以知道如下内容：

①表的读取顺序。（对应id）

②数据读取操作的操作类型。（对应select_type）

③哪些索引可以使用。（对应possible_keys）

④哪些索引被实际使用。（对应key）

⑤表直接的引用。（对应ref）

⑥每张表有多少行被优化器查询。（对应rows）

#### 3.3 explain包含的信息

explain使用：explain+sql语句，通过执行explain可以获得sql语句执行的相关信息。

![](/image/mysql/image-20200303231059955.png)

下面对explain的表头字段含义进行解释：

```
（1）select_type行指定所使用的SELECT查询类型，这里值为SIMPLE，表示简单的SELECT，不使用UNION或子查询。其他可能的取值有PRIMARY、UNION、SUBQUERY等。

（2）table行指定数据库读取的数据表的名字，它们按被读取的先后顺序排列。

（3）type行指定了本数据表与其他数据表之间的关联关系，可能的取值有system、const、eq_ref、ref、range、index和All。

（4）possible_keys行给出了MySQL在搜索数据记录时可选用的各个索引。

（5）key行是MySQL实际选用的索引。

（6）key_len行给出索引按字节计算的长度，key_len数值越小，表示越快。

（7）ref行给出了关联关系中另一个数据表里的数据列名。

（8）rows行是MySQL在执行这个查询时预计会从这个数据表里读出的数据行的个数。

（9）Extra行提供了与关联操作有关的信息。
```



#### 3.4 各个字段解释

​     注意：笔者使用的测试数据库为mysql官方提供的employees数据库。

##### 3.4.1 id

​         select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序，共有三种情况：

​    情况一：id相同，执行顺序由上至下

```sql
explain
select *
from employees t1,
     dept_emp t2,
     salaries t3
where t1.emp_no = t2.emp_no
  and t1.emp_no = t3.emp_no
  and t1.last_name = 'Facello';
```

![image-20200304234927744](/image/mysql/image-20200304234927744.png)

​    情况二：id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

```sql
explain
select *
from employees t1
where t1.emp_no = (select emp_no
                   from dept_emp t2
                   where t2.emp_no = (select emp_no
                                      from salaries t3
                                      where t3.salary = '109964'
                   ));
```

![image-20200305000232154](/image/mysql/image-20200305000232154.png)

​    情况三： id 相同不同，同时存在

```sql
explain
select *
from employees t1,
     (select emp_no
      from dept_emp t2
      where t2.emp_no = (select emp_no
                         from salaries t3
                         where t3.salary = '109964'
      )) s1
where t1.emp_no = s1.emp_no;
```

![image-20200306213513233](/image/mysql/image-20200306213513233.png)

id相同的，可以认为是一组，从上往下顺序执行；在所有组中，id越大优先级越高，越先执行。

##### 3.4.2  select_type

查询的类型，主要用于区别普通查询、联合查询、子查询等的复杂查询；

有如下枚举值：

```
1. SIMPLE：简单的查询，没有子查询和UNION 
2. PRIMARY： 如果有复杂查询的话，标记最外层的查询为primary
3. UNION：若第二个SELECT出现在UNION之后，则被标记为UNION;若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
4. DEPENDENT  UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询 
5. UNION RESULT：UNION的结果
6. SUBQUERY:子查询中的第一个SELECT 
7. DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询 
8. DERIVED：在FROM列表中包含的子查询被标记为DERIVED（衍生）,MySQL会递归执行这些子查询，把结果放在临时表里。
9. MATERIALIZED：具体化的子查询
10. UNCACHEABLE SUBQUERY ：子查询的结果不能被缓存,必须重新评估为每一行的外部查询 
11. UNCACHEABLE UNION：UNION中的第二个或后面的SELECT语句，而且不能被缓存
```

##### 3.4.3 partitions

​     查询将从中匹配记录的分区。对于非分区表，该值为NULL。

##### 3.4.4 table

​     显示这一行的数据是关于哪张表的

##### 3.4.5 type

显示查询使用了何种类型,从最好到最差依次是：

```sql
system>const>eq_ref>ref>fulltext>range>ref_or_null>index>ALL
```

```
- system:表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个也可以忽略不计
- const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键至于where列表中，MySQL就能将该查询转换为一个常量
- eq_ref：唯一性索引，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描
- ref : 非唯一索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。
- fulltext:使用fulltext index
- ref_or_null：类似于ref,会额外搜索包含NULL的行，常用于解析子查询
- index_merge：使用了索引合并优化，在输出key列上包含有多个index.
- range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引,一般就是在你的where语句中出现了=, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, or IN() 等的查询.ß这种范围扫描索引扫描比全表扫描要好，因为他只需要开始索引的某一点，而结束语另一点，不用扫描全部索引。
- index：扫描整个index tree，与all的区别就是all是全表扫描，index是full index 扫描
- all：全表扫描，实际使用中，如果数据量比较大，应该避免进行全表扫描，因为这种连接是最慢的
  备注：一般来说，得保证查询只是达到range级别，最好达到ref
```



##### 3.4.6 possible_keys

​     显示可能应用在这张表中的索引,一个或多个。
​     查询涉及的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用.

##### 3.4.7  key

实际使用的索引。如果为null则没有使用索引

查询中若使用了覆盖索引，则索引和查询的select字段重叠

##### 3.4.8  key_len

表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.
key_len 的计算规则如下:

- 字符串
  - char(n): n 字节长度
  - varchar(n): 如果是 utf8 编码, 则是 3 *n + 2字节; 如果是 utf8mb4 编码, 则是 4* n + 2 字节.
- 数值类型:
  - TINYINT: 1字节
  - SMALLINT: 2字节
  - MEDIUMINT: 3字节
  - INT: 4字节
  - BIGINT: 8字节
- 时间类型
  - DATE: 3字节
  - TIMESTAMP: 4字节
  - DATETIME: 8字节
- 字段属性: NULL 属性 占用一个字节. 如果一个字段是 NOT NULL 的, 则没有此属性.

##### 3.4.9 ref

​      显示索引哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。

##### 3.4.10  rows

​         根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，数值越小越好

##### 3.4.11 filtered

​     已过滤的列指示将被表条件过滤的表行的估计百分比。最大值为100，这表示未过滤行。值从100减小表示过滤量增加。rows显示检查的估计行数，rows×filtered的行显示将与下表连接的行数。例如，如果行数为1000，过滤条件为50.00（50％），则与下表连接的行数为1000×50％= 500。

##### 3.4.12 Extra

```
1.Using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。
MySQL中无法利用索引完成排序操作成为“文件排序”
2.Using temporary：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by 和分组查询 group by
3.USING index：表示相应的select操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表面索引用来读取数据而非执行查找动作。
4.Using where：表面使用了where过滤
5.using join buffer：使用了连接缓存
6.impossible where：where子句的值总是false，不能用来获取任何元组
7.select tables optimized away：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
8.distinct：优化distinct，在找到第一匹配的元组后即停止找同样值的工作
```

拓展知识：覆盖索引

```
覆盖索引（covering index）指一个查询语句的执行只用从索引中就能够取得，不必从数据表中读取。也可以称之为实现了索引覆盖。
如果一个索引包含了（或覆盖了）满足查询语句中字段与条件的数据就叫做覆盖索引。
当一条查询语句符合覆盖索引条件时，sql只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。
使用覆盖索引Innodb比MyISAM效果更好----InnoDB使用聚集索引组织数据，如果二级索引中包含查询所需的数据，就不再需要在聚集索引中查找了

注：遇到以下情况，执行计划不会选择覆盖查询
1.select选择的字段中含有不在索引中的字段 ，即索引没有覆盖全部的列。
2.where条件中不能含有对索引进行like的操作。
```



### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```



### 888.参考文档

https://dev.mysql.com/doc/refman/8.0/en/explain-output.html

https://www.jianshu.com/p/77eaad62f974



