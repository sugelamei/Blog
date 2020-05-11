---
title: Mysql 8 高级-17 MySql中常用工具
date: 2020-03-27 18:36:26
tags: 
    - Mysql
    - CentOS
typora-root-url: ..
---

### 1. mysql

该mysql不是指mysql服务，而是指mysql的客户端工具。

语法 ：

```
mysql [options] [database]
```

#### 1.1 连接选项

```mysql
参数 ： 
	-u, --user=name			指定用户名
	-p, --password[=name]	指定密码
	-h, --host=name			指定服务器IP或域名
	-P, --port=3306		    指定连接端口

示例 ：
	mysql -h 127.0.0.1 -P 3306 -u root -p
	
	mysql -h127.0.0.1 -P3306 -uroot -p2143
	
```

<!--more-->

#### 1.2 执行选项

```mysql
###执行SQL语句并退出
-e, --execute=name		
```

此选项可以在Mysql客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。

```mysql
##示例：
mysql -uroot -pWxc@1234 staff -e "select * from tb_book";
```

![image-20200412010008943](/image/mysql/17/170001.png) 



### 2.mysqladmin

mysqladmin 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

可以通过 以下指令查看帮助文档

```mysql
mysqladmin --help
```



 

![image-20200412010216692](/image/mysql/17/170002.png) 

```mysql
###示例 ：
mysqladmin -uroot -pWxc@1234 create 'test01';  
mysqladmin -uroot -pWxc@1234 drop 'test01';
mysqladmin -uroot -pWxc@1234 version;
	
```



### 3. mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog 日志管理工具。

语法 ：

```mysql
mysqlbinlog [options]  log-files1 log-files2 ...

##选项：
###指定数据库名称只列出指定的数据库相关操作
-d, --database=name。
### 忽略掉日志中的前n行命令
-o, --offset=n
### 将输出的文本格式日志输出到指定文件。
-r,--result-file=name
### 显示简单格式， 省略掉一些信息
-s, --short-form 
### 指定日期间隔内的所有日志。
--start-datatime=date1 --stop-datetime=date2
###  指定位置间隔内的所有日志。
--start-position=pos1 --stop-position=pos2 
```



### 4. mysqldump

mysqldump 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句。

语法 ：

```mysql
mysqldump [options] db_name [tables]

mysqldump [options] --database/-B db1 [db2 db3...]

mysqldump [options] --all-databases/-A
```

#### 4.1 连接选项

```mysql
参数 ： 
	-u, --user=name			指定用户名
	-p, --password[=name]	指定密码
	-h, --host=name			指定服务器IP或域名
	-P, --port=3306			指定连接端口
```



#### 4.2 输出内容选项

```mysql
##参数：
###在每个数据库创建语句前加上 Drop database 语句
--add-drop-database		
###在每个表创建语句前加上 Drop table 语句 , 默认开启 ; 不开启 (--skip-add-drop-table)
--add-drop-table
###不包含数据库的创建语句
-n, --no-create-db		
###不包含数据表的创建语句
-t, --no-create-info
###不包含数据
-d --no-data	
###自动生成两个文件：一个.sql文件，创建表结构的语句,一个.txt文件，数据文件，相当于select into outfile  
-T, --tab=name
```



### 5. mysqlimport/source

mysqlimport 是客户端数据导入工具，用来导入mysqldump 加 -T 参数后导出的文本文件。

语法：

```mysql
mysqlimport [options]  db_name  textfile1  [textfile2...]
```

如果需要导入sql文件,可以使用mysql中的source 指令 : 

```
source /root/tb_book.sql
```



### 6. mysqlshow

mysqlshow 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

语法：

```mysql
mysqlshow [options] [db_name [table_name [col_name]]]
```

参数：

```mysql
###显示数据库及表的统计信息（数据库，表 均可以不指定）
--count		
###显示指定数据库或者指定表的状态信息
-i			
```

示例：

```mysql
#查询每个数据库的表的数量及表中记录的数量
mysqlshow -uroot -pWxc@1234 --count

#查询xxl_job库中每个表中的字段书，及行数
mysqlshow -uroot -pWxc@1234 xxl_job --count

#查询staff库中book表的详细情况
mysqlshow -uroot -pWxc@1234 xxl_job superdevops --count

```

![image-20200412011524301](/image/mysql/17/170003.png)

![image-20200412011633635](/image/mysql/17/170004.png)

![image-20200412011947313](/image/mysql/17/170005.png)

### 666. 彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 





关注博客：

```
 http://superdevops.cn
```



