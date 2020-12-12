---
title: InnoDB的一条记录长什么样子
date: 2020-09-24 15:36:26
categories:
- [MySQL, InnoDB]
tags:
- MySQL
- InnoDB
---

InnoDB的一条记录长什么样子

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028143018.png)

<!-- more -->

## 前言

前文介绍了 MySQL 的基本架构，大体上可以分为「Server 层」和「存储引擎层」两部分。

其中存储引擎包括多种，它们存储数据的形式也不尽相同。有的存储引擎（比如 Memory 引擎）只把数据存储在内存、并不会持久化到磁盘，这样一旦服务器挂了，数据就没了。而默认的 InnoDB 引擎则会把数据持久化到磁盘中。

那么，我们插入的一条记录在 InnoDB 引擎中以什么格式存储的呢？本文进一步介绍和分析。

## InnoDB 记录格式

### 页

在分析 InnoDB 的行记录格式前，先简单介绍下「页」的概念。

InnoDB 将存储的数据划分为若干个「页」，以页作为磁盘和内存交互的基本单位，一个页的大小默认为 16KB。可以通过下面命令查看默认页的大小（单位是字节）：

```sql
mysql> show status like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.01 sec)
```

也就是说，即便我们只查询一条记录，InnoDB 也会把至少 16KB 的内容从磁盘读到内存中。

### 记录格式

InnoDB 的行格式有四种，分别是 Compact、Redundant、Dynamic 和 Compressed，它们在原理上大体都是相同的。本文主要分析 Compact 格式。

可以使用下面命令查看默认行格式（此处 MySQL 版本为 5.7）：

```sql
mysql> show variables like 'innodb_default_row_format';
+---------------------------+---------+
| Variable_name             | Value   |
+---------------------------+---------+
| innodb_default_row_format | dynamic |
+---------------------------+---------+
1 row in set (0.00 sec)

# 或者
mysql> SELECT @@innodb_default_row_format;
+-----------------------------+
| @@innodb_default_row_format |
+-----------------------------+
| dynamic                     |
+-----------------------------+
1 row in set (0.00 sec)
```

也可以在建表时指定行格式，或者建表后再修改。

Compact 行格式的结构大概如图所示：

![img](https://gitee.com/wuwenlun/img-bed/raw/master/img/20200924154248.jpg)

主要分为两部分：记录的额外信息和记录的真实数据。下面分别介绍。

### 额外信息

为便于举例分析，这里先建一个表如下：

```sql
mysql> show create table t1\G;
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` varchar(10) DEFAULT NULL,
  `c2` varchar(10) NOT NULL,
  `c3` char(10) DEFAULT NULL,
  `c4` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

#### 变长字段长度列表

- 概念说明

MySQL 中有些类型的字段长度是不固定的，比如 VARCHAR(M) 类型、TEXT 等，这就导致每条记录中该字段的「实际」长度可能是不一样的。

为此，MySQL 在存储这些变长类型的数据时，实际上分成了两部分存储，分别是：

1. 真实的数据
2. 数据占用的字节数

其中数据占用的字节数就保存在「变长字段长度列表」中。它是以列的「逆序」存储表中变长字段的实际长度的。

- 举例分析

以上面的 t1 表为例，它的变长字段为 c1、c2、c4，若有一条数据如下：

```sql
mysql> select * from t1;
+------+----+------+------+
| c1   | c2 | c3   | c4   |
+------+----+------+------+
| a    | bb | ccc  | dddd |
+------+----+------+------+
1 row in set (0.00 sec)
```

则这三个字段占用的字节数分别为：1、2、4，以「逆序」存放在变长字段长度列表为 040201（十六进制）。

> PS: 字节数跟字符集有关系，latin1 字符默认占用一个字节。 

#### NULL 值列表

- 概念说明

MySQL 中有些列是允许为 NULL 的，如果这些列很多、每个 NULL 值都在表中存储的话会很占用空间。Compact 把这些 NULL 统一管理了起来，放到了 NULL 值列表。

它的处理过程如下：

1. 统计表中允许为 NULL 的列

2. 1. 若列都不允许为 NULL，则 NULL 值列表就不存在了；
   2. 否则，以一个「二进制位」来表示一个允许为空的列，仍是「逆序」排列，其中 1 表示 NULL，0 表示非 NULL 

3.  若 NULL 值列表不足整数「字节」，在高位补 0

- 举例分析

以上面 t1 表为例，c1、c3、c4 三列都允许为 NULL，则使用 3 位表示三个允许为空的列，不足一个字节（8 位），因此高 5 位补 0，如图所示：

![img](https://gitee.com/wuwenlun/img-bed/raw/master/img/20200924154247.jpg)

再插入一条数据（'a', 'bb', NULL, NULL）：

```sql
mysql> select * from t1;
+------+----+------+------+
| c1   | c2 | c3   | c4   |
+------+----+------+------+
| a    | bb | ccc  | dddd |
| a    | bb | NULL | NULL |
+------+----+------+------+
2 rows in set (0.00 sec)
```

由于第一条记录的字段都不是 NULL，因此它的 NULL 值列表为：0000 0000；

第二条记录的 c3、c4 列为 NULL，它的 NULL 值列表为：0000 0110。

#### 记录头信息

第三部分是记录头信息（个人觉得有点类似 JVM 中的对象头信息），该部分占用 5 个字节，示意图如下：

![img](https://gitee.com/wuwenlun/img-bed/raw/master/img/20200924154249.jpg)

其中各个部分大概说明如下：

- 前两个预留位：暂无用处（各占 1 位）
- delete_mask：1 位，标记该条记录是否被删除
- min_rec_mask：1位，B+树每层非叶子节点中的最小记录都会添加该标记
- n_owned：4 位，当前记录拥有的记录数
- heap_no：13 位，当前记录在记录堆的位置
- record_type：3 位，记录的类型，记录分为多种类型，使用该位做区分
- next_record：16 位，保存下一条记录的相对位置

这些东西似乎有点多，其中大部分跟索引和页有关，后文再介绍页的时候再分析。

### 真实数据

总算到了真实的数据部分，但这部分其实也并非只有我们自定义的列，大体可分为两部分：隐藏列和自定义数据。

#### 隐藏列

这部分是 InnoDB 默认添加的列。主要包括三部分：

##### row_id

真实名称为 DB_ROW_ID，表示行记录的唯一标识，这一列并不是必须的。

说起 row_id，有必要提一下 InnoDB 的主键生成策略，它遵循如下顺序：

1. 优先使用用户定义的主键
2. 若未定义主键，则选取唯一键作为主键
3. 若无唯一键，添加 row_id 作为主键

即，当我们新建一个表时，若没有指定主键（Primary Key），InnoDB 会选择一个唯一键（Unique Key）作为主键，如果表中唯一键也没定义，则就要添加一个 row_id 来充当主键了。因为还要自己生成 id，这样会降低效率。

阿里巴巴的《Java开发手册》中有一条相关规定：

![img](https://gitee.com/wuwenlun/img-bed/raw/master/img/20200924154250.jpg)

##### transaction_id & roll_pointer

transaction_id 的真实名称为 DB_TRX_ID，表示事务的 id；roll_pointer 真实名称为 DB_ROLL_PTR，表示回滚指针。

这二者都跟事务密切相关，而且都是必须的，后面再进行分析。

### 我们自定义的数据列

这部分才是我们真正的自定义的列的数据，以 t1 表为例，就是我们定义的 c1、c2、c3、c4 这四列的数据，不再赘述。



## 参考资料

* [https://zhuanlan.zhihu.com/p/147387036](https://zhuanlan.zhihu.com/p/147387036)