---
title: Mysql高级-01 架构介绍 
date: 2020-02-25  17:55:26
tags: 
    - Mysql
    - CentOS
---

### 1.`Mysql`逻辑架构介绍 

​     和其他数据库相比，mysql有点与众不同，它的架构可以在多种不同的场景中应用并发挥良好的作用，主要体现在储存引擎的架构上，插拔式的储存引擎将处理和其他的系统任务以及数据的存储提取相分离。这种架构可以根据业务的需要和实际需要选择合适的存储引擎。

![](/image/mysql/image-20200229101759889.png)

从上图可知，MySQL的逻辑框架主要分为四层：

<!-- more -->

#### 1.1 连接层

   最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务段工具实现的类似于Tcp/IP的通信。主要完成一些类似于连接处理，授权认证以及相关的安全方案。在该层上引入了线程池的概念，通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全连接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

>   Connectors：指的是不同语言中与SQL的交互

#### 1.2 服务层 

  第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化以及部分内置函数的执行。所有跨存储引擎的功能都在这一层实现，如过程，函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成的优化，如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会存储内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

> Management Serveices & Utilities：系统管理和控制工具
>
> Connection Pool: 连接池
>
> - 管理缓冲用户连接，线程处理等需要缓存的需求。
> - 负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。
>
> SQL Interface: SQL接口。
>
> - 接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface。
>
> Parser: 解析器。
>
> - SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。在 MySQL中我们习惯将所有 Client 端发送给 Server 端的命令都称为 query ，在 MySQL Server 里面，连接线程接收到客户端的一个 Query 后，会直接将该 query 传递给专门负责将各种 Query 进行分类然后转发给各个对应的处理模块。主要功能：
>   - 将SQL语句进行语义和语法的分析，分解成数据结构，然后按照不同的操作类型进行分类，然后做出针对性的转发到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。
>   - 如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的。
>
> Optimizer: 查询优化器。
>
> - SQL语句在查询之前会使用查询优化器对查询进行优化。就是优化客户端请求的 query（sql语句） ，根据客户端请求的 query 语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个 query 语句的结果，它使用的是“选取-投影-联接”策略进行查询。
> - 用一个例子就可以理解： select id,name from user where gender = 1;这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤, 这个select查询先根据id和name进行属性投影，而不是将属性全部取出以后再进行过滤,将这两个查询条件联接起来生成最终查询结果.
> 
> Cache&Buffer： 查询缓存。
> 
>-   它的主要功能是将客户端提交 给MySQL 的 Select 类 query 请求的返回结果集 cache 到内存中，与该 query 的一个 hash 值 做一个对应。该 Query 所取数据的基表发生任何数据的变化之后， MySQL 会自动使该 query 的Cache 失效。在读写比例非常高的应用系统中， Query Cache 对性能的提高是非常显著的。当然它对内存的消耗也是非常大的。
> - 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等
>
> 

#### 1.3 引擎层

​     储存引擎层是真正负责了mysql中数据的储存和提取，服务器通过分析API和存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。我们最常用的存储引擎主要是InnoDB和MyISAM。

> Storege Engines存储引擎接口
>
> - 存储引擎接口模块可以说是 MySQL 数据库中最有特色的一点了。目前各种数据库产品中，基本上只有 MySQL 可以实现其底层数据存储引擎的插件式管理。这个模块实际上只是 一个抽象类，但正是因为它成功地将各种数据处理高度抽象化，才成就了今天 MySQL 可插拔存储引擎的特色。
> - MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者都可以按照自己的意愿来进行开发。
> - 注意：存储引擎是基于表的，而不是数据库。

#### 1.4储存层

​     数据储存层主要是将数据储存在运行于裸设备的文件系统之上并完成与存储引擎的交互。

### 2.`Mysql`存储引擎

   1.直接通过show engines命令可以查看MySQL支持的存储引擎。

```sql
show engines;
```

![image-20200229111136954](/image/mysql/image-20200229111136954.png)   



​    2.通过show variables like '%storage_engine%'查看MySQL的当前默认存储引擎。

```sql
show variables like '%storage_engine%';
```

![image-20200229111338557](/image/mysql/image-20200229111338557.png)

   3.MyISAM和InnoDB进行比较，主要区别如下表：

![](/image/mysql/image-20200229111338558.png)

注意：MyISAM主要关注性能，其查询速度快。

​           InnoDB主要关注高并发，其支持事务。 

### 3.`Mysql`配置文件   

​          构成MySQL数据库和InnoDB存储引擎表的各种类型文件：

#### 3.1参数文件 

​       告诉MySQL实例启动时在哪里可以找到数据库文件，并且指定某些初始化参数，这些参数定义了某种内存结构的大小等设置，还会介绍各种参数的类型。

​     我们可以使用以下命令显示所有的参数：

```sql
show variables;
```

​     我们可以使用以下命令显示指定的参数：

```sql
show variables like '%你想要查询的参数%'；
```

​    我们也在 `/etc/my.cnf`下修改相应的参数。

#### 3.2 日志文件

​     用来记录MySQL实例对某种条件做出响应时写入的文件，如错误日志文件、二进制日志文件、慢查询日志文件、查询日志文件等

##### 3.2.1 错误日志

错误日志文件对MySQL的启动、运行、关闭过程进行了记录。MySQL DBA在遇到问题时应该首先查看该文件以便定位问题。该文件不仅记录了所有的错误信息，也记录一些警告信息或正确的信息。

用户可以通过命令来定位该文件:

```
show variables like 'log_error';
```

![image-20200229113303192](/image/mysql/image-20200229113303192.png)

##### 3.2.2 慢查询日志

​     慢查询日志（slow log）可帮助DBA定位可能存在问题的SQL语句，从而进行SQL语句层面的优化。例如，可以在MySQL启动时设一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询日志文件中。DBA每天或每过一段时间对其进行检查，确认是否有SQL语句需要进行优化。该阈值可以通过参数long_query_time来设置，默认值为10，代表10秒。

在默认情况下，MySQL数据库并不启动慢查询日志，用户需要手工将这个参数设为ON（临时）：

```
SET GLOBAL slow_query_log=ON;
```

查看慢查询阈值：

```
show variables like 'long_query_time';
```

![image-20200229115503530](/image/mysql/image-20200229115503530.png)

​    设置long_query_time这个阈值后，MySQL数据库会记录运行时间超过该值的所有SQL语句，但运行时间正好等于long_query_time的情况并不会被记录下。也就是说，在源代码中判断的是大于long_query_time，而非大于等于。

查看和慢查询相关的信息：

```
show variables like '%slow_query%';
```

![image-20200229115717928](/image/mysql/image-20200229115626315.png)

​     一个和慢查询日志有关的参数是log_queries_not_using_indexes，如果运行的SQL语句没有使用索引，则MySQL数据库同样会将这条SQL语句记录到慢查询日志文件;

手动开启log_queries_not_using_indexes参数（临时）：

```
SET GLOBAL log_queries_not_using_indexes=ON;
```

查看是否开启log_queries_not_using_indexes参数

```
show variables like  'log_queries_not_using_indexes';
```

![image-20200229122505363](/image/mysql/image-20200229122505363.png)

  参数log_throttle_queries_not_using_indexes，用来表示每分钟允许记录到slow log的且未使用索引的SQL语句次数。该值默认为0，表示没有限制。在生产环境下，若没有使用索引，此类SQL语句会频繁地被记录到slow log，从而导致slow log文件的大小不断增加;

```
show variables like 'log_throttle_queries_not_using_indexes';
```

![image-20200229122947288](/image/mysql/image-20200229122947288.png)

慢查询的日志记录放入一张表中，这使得用户的查询更加方便和直观。慢查询表在mysql架构下，名为slow_log，其表结构定义如下：

```
show create table  mysql.slow_log;
```

![image-20200229123515019](/image/mysql/image-20200229123515019.png)

参数log_output指定了慢查询输出的格式，默认为FILE:

```
show variables like 'log_output';
```

![image-20200229210640763](/image/mysql/image-20200229210640763.png)

可以将它设为TABLE,然后就可以查询mysql架构下的slow_log表.

```
SET GLOBAL log_output='TABLE';
```



##### 3.2.3 查询日志

​        查询日志记录了所有对MySQL数据库请求的信息，无论这些请求是否得到了正确的执行。默认文件名为：主机名.log。

##### 3.2.4 二进制日志

​      二进制日志（binary log）记录了对MySQL数据库执行更改的所有操作，但是不包括SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改。然而，若操作本身并没有导致数据库发生变化，那么该操作可能也会写入二进制日志。

​      总的来说，二进制日志主要有以下几种作用：

- 恢复（recovery）：某些数据的恢复需要二进制日志，例如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行point-in-time的恢复。
- 复制（replication）：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySQL数据库（一般称为slave或standby）与一台MySQL数据库（一般称为master或primary）进行实时同步。
-  审计（audit）：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入的攻击。

  通过配置参数log-bin[=name]可以启动二进制日志。如果不指定name，则默认二进制日志文件名为主机名，后缀名为二进制日志的序列号，所在路径为数据库所在目录（datadir）；

```
show variables like 'datadir';
```

![image-20200229212642402](/image/mysql/image-20200229212642402.png)

​         二进制日志文件在默认情况下并没有启动，需要手动指定参数来启动。可能有人会质疑，开启这个选项是否会对数据库整体性能有所影响。不错，开启这个选项的确会影响性能，但是性能的损失十分有限。根据MySQL官方手册中的测试表明，开启二进制日志会使性能下降1%。但考虑到可以使用复制（replication）和point-in-time的恢复，这些性能损失绝对是可以且应该被接受的。



以下配置文件的参数影响着二进制日志记录的信息和行为：

1.max_binlog_size

​        指定单个二进制日志文件的最大值,如果超过该值，则产生新的二进制日志文件，后缀名+1，并记录到.index文件;

2.binlog_cache_size

​           当使用事务的表存储引擎（如InnoDB存储引擎）时，所有未提交（uncommitted）的二进制日志会被记录到一个缓存中去，等该事务提交（committed）时直接将缓冲中的二进制日志写入二进制日志文件，而该缓冲的大小由binlog_cache_size决定，默认大小为32K;binlog_cache_size是基于会话（session）的，也就是说，当一个线程开始一个事务时，MySQL会自动分配一个大小为binlog_cache_size的缓存，因此该值的设置需要相当小心，不能设置过大。当一个事务的记录大于设定的binlog_cache_size时，MySQL会把缓冲中的日志写入一个临时文件中，因此该值又不能设得太小。通过SHOWGLOBAL STATUS命令查看binlog_cache_use、binlog_cache_disk_use的状态，可以判断当前binlog_cache_size的设置是否合适。binlog_cache_use记录了使用缓冲写二进制日志的次数，binlog_cache_disk_use记录了使用临时文件写二进制日志的次数。

3.sync_binlog

​      参数sync_binlog=[N]表示每写缓冲多少次就同步到磁盘。如果将N设为1，即sync_binlog=1表示采用同步写磁盘的方式来写二进制日志，这时写操作不使用操作系统的缓冲来写二进制日志。sync_binlog的默认值为0，如果使用InnoDB存储引擎进行复制，并且想得到最大的高可用性，建议将该值设为ON。不过该值为ON时，确实会对数据库的IO系统带来一定的影响。

4.binlog-do-db/binlog-ignore-db

   参数binlog-do-db和binlog-ignore-db表示需要写入或忽略写入哪些库的日志。默认为空，表示需要同步所有库的日志到二进制日志。



5.log-slave-update

​      如果当前数据库是复制中的slave角色，则它不会将从master取得并执行的二进制日志写入自己的二进制日志文件中去。如果需要写入，要设置log-slave-update。如果需要搭建master=>slave=>slave架构的复制，则必须设置该参数。

6.binlog_format

​     binlog_format参数，该参数可设的值有STATEMENT、ROW和MIXED。

​    STATEMENT:二进制日志文件记录的是日志的逻辑SQL语句。

​     ROW:二进制日志记录的不再是简单的SQL语句了，而是记录表的行更改情况。

​     MIXED:MySQL默认采用STATEMENT格式进行二进制日志文件的记录，但是在一些情况下会使用ROW格式。

#### 3.3 socket文件

当用UNIX域套接字方式进行连接时需要的文件。

套接字文件可由参数socket控制。一般在/tmp目录下，名为mysql.sock：

```
show variables like 'socket';
```

![image-20200229214012085](/image/mysql/image-20200229214012085.png)

#### 3.4 pid文件

MySQL实例的进程ID文件。当MySQL实例启动时，会将自己的进程ID写入一个文件中——该文件即为pid文件。该文件可由参数pid_file控制，默认位于数据库目录下，文件名为主机名.pid

```
show variables like 'pid_file';
```

![image-20200229214130420](/C:/Users/love/AppData/Roaming/Typora/typora-user-images/image-20200229214130420.png)



#### 3.5 MySQL表结构文件

用来存放MySQL表结构定义文件。

#### 3.6 存储引擎文件

因为MySQL表存储引擎的关系，每个存储引擎都会有自己的文件来保存各种数据。这些存储引擎真正存储了记录和索引等数据。主要介绍与InnoDB有关的存储引擎文件。

##### 3.6.1  表空间文件

​      InnoDB采用将存储的数据按表空间（tablespace）进行存放的设计。在默认配置下会有一个初始大小为10MB，名为ibdata1的文件。该文件就是默认的表空间文件（tablespace file），用户可以通过参数innodb_data_file_path对其进行设置，格式如下：

```
innodb_data_file_path=datafile_spec1[;dadatafile_spec2]
```

​      设置innodb_data_file_path参数后，所有基于InnoDB存储引擎的表的数据都会记录到该共享表空间中。若设置了参数innodb_file_per_table，则用户可以将每个基于InnoDB存储引擎的表产生一个独立表空间。独立表空间的命名规则为：表名.ibd。通过这样的方式，用户不用将所有数据都存放于默认的表空间中。

##### 3.6.2 重做日志文件

​      在默认情况下，在InnoDB存储引擎的数据目录下会有两个名为ib_logfile0和ib_logfile1的文件。在MySQL官方手册中将其称为InnoDB存储引擎的日志文件，不过更准确的定义应该是重做日志文件（redo log file）。

下列参数影响着重做日志文件的属性：

​      innodb_log_file_size:参数innodb_log_file_size指定每个重做日志文件的大小。

​      innodb_log_files_in_group:参数innodb_log_files_in_group指定了日志文件组中重做日志文件的数量，默认为2。

​     innodb_log_group_home_dir：参数innodb_log_group_home_dir指定了日志文件组所在路径，默认为./

​      重做日志文件的大小设置对于InnoDB存储引擎的性能有着非常大的影响。一方面重做日志文件不能设置得太大，如果设置得很大，在恢复时可能需要很长的时间；另一方面又不能设置得太小了，否则可能导致一个事务的日志需要多次切换重做日志文件。此外，重做日志文件太小会导致频繁地发生async checkpoint，导致性能的抖动。

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```



