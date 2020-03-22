---
title: Mysql 8 高级-07 索引分析
date: 2020-03-05  21:40:36
tags: 
    - Mysql
    - CentOS
---

### 1.单表分析

#### 1.1准备数据

```sql
create table if not exists article(
  id int primary key auto_increment,
  author_id int not null,
  category_id int not null,
  views int not null,
  comments int not null,
  title varchar(255) not null,
  content text not null
);

insert into article values(null,1,1,1,1,'1','1');
insert into article values(null,2,2,2,2,'2','2');
insert into article values(null,1,1,3,3,'3','3');

```

<!--more--->

#### 1.2 使用explain分析

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views

explain select  id,author_id,views  from  article
where category_id = 1
and comments > 1;
```

![image-20200308204926030](/image/mysql/image-20200308164243654.png)

分析：type=ALL，产生了全表扫描。

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views
-- 并且找出views查看次数最多的记录
explain select  id,author_id,views from  article
where category_id = 1
and comments > 1
order by views desc limit 1;
```

![image-20200308205252700](/image/mysql/image-20200308193754271.png)

分析：type=ALL,产生了全表扫描, 并且出现了Using filesort,使用了外部的索引排序。

#### 1.3 建立索引

```sql
create index ind_article_ccv on article(category_id,comments,views);
```

#### 1.4 再次使用explain分析

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views

explain select  id,author_id,views  from  article
where category_id = 1
and comments > 1;
```

![image-20200308205751539](/image/mysql/image-20200308205751539.png)

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views
-- 并且找出views查看次数最多的记录
explain select  id,author_id,views from  article
where category_id = 1
and comments > 1
order by views desc limit 1;
```

![image-20200308205924276](/image/mysql/image-20200308205924276.png)

分析：创建索引之后type=range, 但是Using filesort 依然存在.

​           索引创建了,为什么在排序的时候没有生效?

​            这是因为先排序category_id, 如果遇到相同的category_id,则再排序comments, 如果遇到相同的comments则再排序views,当comments字段在联合索引处于中间位置时，因为comments>1条件是一个范围值,所以type=range，mysql无法再利用索引对后面的views部分进行检索,即range类型查询字段后面的索引全都无效

综上所述: 索引创建有问题

#### 1.5 删除上面创建的索引

```sql
-- 删除上面创建的索引:
drop index ind_article_ccv on article;
```

#### 1.6 重新创建索引

```sql
-- 重新创建索引: 
create index ind_art_cb on article(category_id,views);
```

#### 1.7 第三次使用explain分析

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views

explain select  id,author_id,views  from  article
where category_id = 1
and comments > 1;
```

![image-20200308211238886](/image/mysql/image-20200308211238886.png)

```sql
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views
-- 并且找出views查看次数最多的记录
explain select  id,author_id,views from  article
where category_id = 1
and comments > 1
order by views desc limit 1;
```

![image-20200308211546458](/image/mysql/image-20200308211546458.png)

分析：type =ref，但是有出现了Backward index scan（反向扫描），说明创建索引的排序顺序不对，应该创建降序索引，默认为升序索引。

#### 1.9 第二次删除索引

```sql
-- 删除上面创建的索引:
drop index ind_art_cb on article;
```

#### 1.10 第三次创建索引

```sql
-- 重新创建索引: 
create index ind_art_cb on article(category_id desc,views desc);
```

#### 1.11  第四次使用explain分析

```
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views

explain select  id,author_id,views  from  article
where category_id = 1
and comments > 1;
```

![image-20200308212723352](/image/mysql/image-20200308212723352.png)

```
-- 查询category_id为1且comments 大于 1 的所有记录,显示id,author_id,views
-- 并且找出views查看次数最多的记录
explain select  id,author_id,views from  article
where category_id = 1
and comments > 1
order by views desc limit 1;
```

![image-20200308212746451](/image/mysql/image-20200308212746451.png)

最终终于优化好了。

### 2.两张表分析

#### 2.1数据准备

```sql
create table if not exists class(
    id int(10) primary key auto_increment,
      card int not null
);

create table  if not exists book(
    bookid int primary key auto_increment,
      card int not null
);

insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));
insert into class(card) values(floor(1+rand()*20));

insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
insert into book(card) values(floor(1+rand()*20));
```

#### 2.2 使用explain分析LEFT JOIN

```sql
-- 下面开始分析
explain select * from class left join book on class.card = book.card;
```

![image-20200308213330294](/image/mysql/image-20200308213330294.png)

  分析： type =ALL， Extra 出现了Using join buffer (Block Nested Loop)；

#### 2.3 创建索引

```sql
create index X on book(card);
```

#### 2.4 再次使用explain分析LEFT JOIN

```sql
explain select * from class left join book on class.card = book.card;
```

![image-20200308214710794](/image/mysql/image-20200308214710794.png)

分析：我们可以看到第二行的type变为了ref,rows也变小了,优化效果明显

#### 2.5  使用explain分析RIGHT JOIN

```sql
explain select * from class right join book on class.card = book.card;
```

![image-20200308215416653](/image/mysql/image-20200308215416653.png)

#### 2.6 删除索引

```sql
drop index X on book;
```

#### 2.7 创建索引

```sql
create index Y on class(card);
```

#### 2.8  再次使用explain分析RIGHT JOIN

```sql
explain select * from class right join book on class.card = book.card;
```

![image-20200308220023970](/image/mysql/image-20200308220023970.png)

#### 2.9创建索引

```sql
create index X_X on book(card);
```

#### 2.10 第三次使用explain分析LEFT JOIN

```sql
explain select * from class left join book on class.card = book.card;
```

![image-20200308220502310](/image/mysql/image-20200308220502310.png)

#### 2.11 第三次使用explain分析RIGHT JOIN

```sql
explain select * from class right join book on class.card = book.card;
```

![image-20200308220256898](/image/mysql/image-20200308220256898.png)



> 总结：在left join中，左表建立索引不起作用，虽然用到索引，但是rows条数并没有变；
>
> ​            在right join中，右表建立索引不起作用，虽然用到索引，但是rows条数并没有变；
>
> 分析原因：left join条件用于确定如何从右表搜索行,左边一定都有；
>
> ​                    right join条件用于确定如何从左表搜索行,右边一定都有；
>
> 结论：left join（左连接）：右表创建索引。
>
> ​            right join（右连接）：左表创建索引。
>
> 简记：左右外连接，索引相反建（left：右表建，right：左表建）。



### 3.三张表优化

#### 3.1 准备数据

```sql
create table if not exists phone(
    phoneid int primary key auto_increment,
      card int
);

insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
insert into phone(card) values(floor(1+rand()*20));
```

#### 3.2 删除索引

```sql
drop index Y on class;
drop index X_X on book;
```

#### 3.3 使用explain分析

```sql
explain select * from class left join book on class.card = book.card left join phone on book.card = phone.card;
```

![image-20200308221753135](/image/mysql/image-20200308221753135.png)

#### 3.4  建立索引

```sql
create index Y on book(card);
create index Z on phone(card);
```

#### 3.5 再次使用explain分析

![image-20200308221933561](/image/mysql/image-20200308221933561.png)

> 分析：后2行的type都是ref且总rows数量大大降低了,效果不错,因此索引最好是设置在需要经常查询的字段中
>
> 结论：尽可能减少join语句中的循环总次数: 永远用小结果集驱动大的结果集;
>            优先优化内层循环；
>            保证join语句中被驱动表上的join条件字段已经创建过索引；
>            当无法保证被驱动表的join条件字段是否有索引且内存资源充足的前提下,不要太吝惜Join Buffer的设置；



### 666.彩蛋

请大家持续关注公众号：Java橙序猿

 ![](/image/common/superdevops.jpg) 

关注博客：

```
 http://superdevops.cn
```

### 888.参考文档

https://www.cnblogs.com/gdwkong/articles/8505125.html