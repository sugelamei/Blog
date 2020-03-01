---
title: Mysql高级-02 SQL语句的执行顺序
date: 2020-02-29  22:36:26
tags: 
    - Mysql
    - CentOS
---

### 0.说在前面

```
- Mysql：8.0.19
- 系统：Centos 7.6.1810
- IDE的选择：选择一款强大的IDE往往能够事半功倍，笔者使用DataGrip；
```

### 1.手写SQL

```sql
select distinct
     <select_list>
from
     <left_table> <join_type>
join <right_table> on <join_condition>
where 
     <where_condition>
group by
     <group_by_list>
having
     <having_confition>
order by
     <order_by_condition>
limit <limit_number>
```

<!--more-->

### 2.机读顺序

#### 2.1文字表示

```sql
1.from <left_table>
2.on <join_condition>
3.<join_type>join <right_table> 
4.where <where_condition>
5.group by<group_by_list>
6.having<having_confition>
7.select
8.distinct<select_list>
9.order by<order_by_condition>
10.limit <limit_number>
```

#### 2.2 图形表示

![](/image/mysql/image-20200229224750440.png)



### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```
