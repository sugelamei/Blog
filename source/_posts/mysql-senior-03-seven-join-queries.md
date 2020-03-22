---
title: Mysql 8 高级-03 七种join查询
date: 2020-02-29  11:40:36
tags: 
    - Mysql
    - CentOS
---

### 1.`Join`介绍

  Join 是在多表关联查询的一个关键字，Join 按照功能大致分为如下三类：

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

<!--more-->

### 2.数据准备

#### 2.1 创建tb_emp表并插入数据。

```
    DROP TABLE IF EXISTS `tb_emp`;
    CREATE TABLE `tb_emp` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `name` varchar(20) NOT NULL,
      `deptid` int(11) NOT NULL,
      PRIMARY KEY (`id`),
      KEY `idx_tb_emp_name` (`name`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    INSERT INTO `tb_emp`(name,deptid) VALUES ('jack', '1');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('tom', '1');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('tonny', '1');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('mary', '2');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('rose', '2');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('luffy', '3');
    INSERT INTO `tb_emp`(name,deptid) VALUES ('outman', '14');
```

#### 2.2 创建tb_dept表并插入数据。

```
DROP TABLE IF EXISTS `tb_dept`;
    CREATE TABLE `tb_dept` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `deptname` varchar(20) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    INSERT INTO `tb_dept`(deptname) VALUES ('研发');
    INSERT INTO `tb_dept`(deptname) VALUES ('测试');
    INSERT INTO `tb_dept`(deptname) VALUES ('运维');
    INSERT INTO `tb_dept`(deptname) VALUES ('经理');
```

从上表插入的数据可知`outman`是没有对应部门的。

### 3. INNER JOIN

INNER JOIN：取A和B共有的数据，也就是交集。

![img](/image/mysql/image-20200229224750441.png)

```sql
select  * from tb_emp  a inner  join  tb_dept b on  a.deptid =b.id;
```

![image-20200301180022967](/image/mysql/image-20200301180022967.png)

注意：可以看出outman和经理2项，因为这两项和其他的没有交集。



### 4. LEFT JOIN

LEFT JOIN：取A和B共有(交集)和A独有。

LEFT JOIN 会读取左边数据表的全部数据，即便右边表无对应数据。



![img](/image/mysql/image-20200301180022968.png)

```
select  * from tb_emp  a left join tb_dept b on  a.deptid =b.id;
```

![image-20200301181012415](/image/mysql/image-20200301181012415.png)

注意：你会发现是table1和table2的交集+table1的独有。

### 5. RIGHT JOIN

 RIGHT JOIN：B独有和取table1和table2共有(交集)。

LEFT JOIN 会读取右边数据表的全部数据，即便左边表无对应数据。



![img](/image/mysql/image-20200301181012416.png)

```
select  * from tb_emp  a right join tb_dept b on  a.deptid =b.id;
```

![image-20200301181507798](/image/mysql/image-20200301181507798.png)

注意：table2的独有+你会发现是table1和table2的交集。

### 6. A独有

![](/image/mysql/image-20200301181507799.png)

```
select  * from tb_emp  a left join tb_dept b on  a.deptid =b.id where b.id is  null;
```

![image-20200301185312596](/image/mysql/image-20200301185312596.png)

注：参照left join，A独有只是将AB交集部分去掉。

### 7. B独有

![](/image/mysql/image-20200301185312597.png)



```
select  * from tb_emp  a right join tb_dept b on  a.deptid =b.id where  a.id is  null;
```

![image-20200301185603794](/image/mysql/image-20200301185603794.png)

注：参照right join，B独有只是将AB交集部分去掉。

### 8. A ,B独有并集

A、B独有并集，相当于A、B全有去掉AB的共有（交集）。

![](/image/mysql/image-20200301185603795.png)

```
select  * from tb_emp  a left join tb_dept b on  a.deptid =b.id where b.id is  null
union
select  * from tb_emp  a right join tb_dept b on  a.deptid =b.id where  a.id is  null;
```

![image-20200301185910508](/image/mysql/image-20200301185910508.png)



### 9. AB全有（并集）

由于mysql中不支持full outer join，所以这里通过union进行转换。AB并集：AB交集+A独有+B独有。

![](/image/mysql/image-20200301185910509.png)

```
select  * from tb_emp  a left join tb_dept b on  a.deptid =b.id
union
select  * from tb_emp  a right join tb_dept b on  a.deptid =b.id ;
```

![image-20200301190441310](/image/mysql/image-20200301190441310.png)

### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```



