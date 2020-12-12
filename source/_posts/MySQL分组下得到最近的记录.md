---
title: MySQL分组下最近的记录
date: 2020-11-27 16:27:26
categories:
- [MySQL, SQL]
tags:
- MySQL
- SQL
---

找到分组条件下最近的一条数据

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201127163535.png)

<!-- more -->

对于不是很熟悉SQL的同学说，立马可能会想到排序后再`limit 1`就搞定了，看到数据后就开始反思了：“这才一条数据，这不是我想要的效果“。有的同学思前想后会写出来，我相信其他大部分同学肯定写不出来，对于这部分同学你们有福了。看了这个段落，就能解决你们的痛点。

##### 建立一张表

该表主要记录用户购买手机日期。我相信一个人在他的一生中肯定有购买手机的经历，有的人一生可能只有一部手机（当然这不太可能），有的人有多部手机。根据该表找到所有用户购买手机最近的记录。

```sql
create table user_phone (
	id int not null auto_increment comment '主键',
    name varchar(16) comment '姓名',
    phone varchar(16) comment '手机型号',
    buy_time datetime comment '购买日期',
    primary key(id)
);

insert into user_phone (name, phone, buy_time) values ("吴文伦", "天语", "2006-07-12"), ("吴文伦", "诺基亚", "2011-08-23"), ("吴文伦", "中兴", "2015-04-08"), ("吴文伦", "魅族", "2017-12-02"), ("吴文伦", "红米", "2020-05-15");
insert into user_phone (name, phone, buy_time) values ("勒布朗", "大哥大", "1995-09-16"), ("勒布朗", "诺基亚", "2001-03-22"), ("勒布朗", "苹果1代", "2007-06-08"), ("勒布朗", "苹果5代", "2013-11-09");
insert into user_phone (name, phone, buy_time) values ("老于", "土砖", "1994-01-01"), ("老于", "华为", "2003-08-09"), ("老于", "vivo", "2009-07-08");
```

数据列表如下：

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201127172045.png)

如何操作才能找到符合业务要求的数据呢？正确数据如下：

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201127174105.png)

--------------------------------------------------------------------------------华丽的分界线----------------------------------------------------------------------------------------

##### 分组原理

表里面有三个人，故应想到对`name`进行分组，分组后能够得到<font color=red>**子数据列表项**</font>，然后根据聚合函数`max`在<font color=red>**子数据列表项**</font>中找到`buy_time`这一列中最大的日期的那一个值。聚合函数的作用就是在一堆列表数据里的指定列进行聚合运算合成一个值，注意它只针对字段列。![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201208200558.png)

执行该SQL语句会得到如下数据

```sql
select name, max(buy_time) as buy_time from user_phone group by name;
```

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201209195825.png)

但是少了id和phone两个字段，很多人可能会想到这样子做：

```sql
select id, name, phone, max(buy_time) as buy_time from user_phone group by name;
```

那我们执行以下，结果报错了！！

```报错日志
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'wuwenlun.user_phone.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

意思是说`sql_mode`为`only_full_group_by`，字段中只允许展示分组的那些字段，在本例中即为`name`，`id`和`phone`不允许展示。如果需要展示，就需要改`sql_mode`，把`only_full_group_by`去掉。下面的语句中已经没有`only_full_group_by`了，执行它。

```sql
set @@session.sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
```

再执行查询语句，看看结果：

```sql
select id, name, phone, max(buy_time) as buy_time from user_phone group by name;
```

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201209201457.png)

虽然可以执行了，但/并不是我们想要的结果（所有用户最近购买手机的记录），为什么名字为`吴文伦`的这条记录id为1？就得通过分组的运行机理去分析。根据name分组后找到`吴文伦`的第一条记录，它是根据记录在表中顺序找到第一个得到的。所以id为`1`，phone为`天语`是这样子得到的。

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201209202925.png)

所以，明白机理后，要得到正确答案就很容易了，先倒排序再分组就可以了，这样子最近的记录会排在最前面。

```sql
select t.* from (select * from user_phone order by buy_time desc) as t group by t.name;
```

卧槽，咋还是不对呢？没有起到任何效果！这是因为外层查询是group by，mysql会把子查询里的order by当做无效处理了，对group by来说它只关注结果集，跟你排不排序没有关系，所以在sql优化过程中就把子查询排序省略了，毕竟排序是会耗时间和资源的。

那如何让分组group by情况下分组不失效呢？全网几乎一致的说后面补上limit去影响结果集，语句如下：

```sql
select t.* from (select * from user_phone order by buy_time desc limit 9999999) as t group by t.name;
```

可以得到正确输出：

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201212140226.png)

但是我总觉得这种操作有点变态，我想着换一种方法，就是先分组得到用户名和该用户最近购买手机的日期这两个值，以此为查询条件再回表查询。语句如下：

```sql
select t.* from user_phone t where (t.name, t.buy_time) in (select name, max(buy_time) as buy_time from user_phone group by name);
```

依然能够得到正确答案：

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201127174105.png)