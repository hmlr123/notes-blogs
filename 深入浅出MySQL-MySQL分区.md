---
title: 深入浅出MySQL-MySQL分区
date: 2019-04-11 18:33:32
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



分区优化数据库表解决的问题、实现的原理、实现的过程及优点

<!--more-->

# 英语词汇

coalesce 		联合联结



# 为什么分区

## 问题

很多情况下，我们的一个表达到了上万条，甚至过亿，这时候我们查询时，无论有没有索引，消耗的时间都是极大的，占用的内存也是很浪费（一般索引会占一半的文件大小）。使用分区则可以解决这些问题



# 分区表的概念

分区是根据一定的规则，数据库把一个表分解成多个更小的、更容易管理的部分。就访问数据库应用而言，逻辑上就只有一个表或者一个索引，但实际上这**个表可能有N个物理分区对象组成，每个分区都是一个独立的对象**，可以独立处理，可以作为表的一部分进行处理。分区对应用来说是完全透明的，不影响应用的业务逻辑。



判断当前MySQL是否支持分区？

```mysql
show variables like'%partition%' 
```



MySQL支持大部分存储引擎（MyISAM、InnoDB、Memery等），不支持Merge、CSV等。

同一分区表的不同分区存储引擎必须一致，不用分区表可以不同存储引擎。



# 分区表的原理

分区表由多个相关的底层表实现，这些底层表是由句柄对象来表示，可以直接访问分区，**分区表的索引只是在各个底层表上各自加上一个完全相同的索引，从存储引擎上看，分区表和普通表没有什么不同。**



SELECT查询：**打开并锁住所有的底层表**，优化器判断是否会过滤部分分区，然后调用对应的存储引擎接口访问各个分区的数据。

INSERT操作：打开并锁住所有底层表，确定哪个分区接收记录，写入底层表

DELETE操作：打开并锁住所有底层表，确定数据对应的分区，删除操作。

UPDATE操作：打开并锁住所有的底层表，确定更新的分区，取出更新，判断更新后的数据应该存放的分区位置，写入操作。



# 分区的种类

`RANGE`分区：基于属于一个给定**连续区间**的列值，把多行分配给分区。

`LIST`分区：类似于按`RANGE`分区，区别在于`LIST`分区是**基于列值匹配一个离散值集合中的某个值**来进行选择。

`HASH`分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含`MySQL` 中**有效的、产生非负整数值**的任何表达式。

`KEY`分区：类似于按`HASH`分区，区别在于`KEY`分区只支持计算一列或多列，且`MySQL`服务器提供其自身的哈希函数。必须有**一列或多列包含整数值。**



重点：无论哪种分区，要么你分区表上没有主键/唯一键，要么**<span style="color:red">分区表的主键/唯一键都必须包含分区键，也就是说不能使用主键/唯一键字段之外的其它字段分区。</span>**



## `RANGE`分区

* 取值范围将数据分成区，区间连续且不重叠。
* 分区键Null值会按最小值处理

创建：

```mysql
mysql> create table emp(
    -> id int not null,
    -> ename varchar(20),
    -> hired date not null default '1123^12&23',
    -> separated date not null default '1234+12+12',
    -> job varchar(30) not null,
    -> store_id int not null)
    -> partition by range(store_id)(
    -> partition p0 values less than (10),
    -> partition p1 values less than (20),
    -> partition p2 values less than (30)
    -> );
Query OK, 0 rows affected (0.19 sec)
```

修改：

```mysql
mysql> alter table emp add partition (partition p3 values less than maxvalue);
Query OK, 0 rows affected (0.41 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除：

```mysql
alter table emp drop partition p0;
```





## `LIST`分区

```mysql
mysql>  create table expenses(
    -> expense_date date not null,
    -> category int,
    -> amount decimal (10,3)
    -> )
    -> partition by list(category)(
    -> partition p0 values in (3,5),
    -> partition p1 values in (1,10),
    -> partition p2 values in (4,9),
    -> partition p3 values in (2),
    -> partition p4 values in (6)
    -> );
Query OK, 0 rows affected (0.20 sec)
```





## `Columns`分区

**解决`list、range`分区只支持整数的问题。**

* `columns`分区分为 `range columns`（基于**元组的数据比较**）和`list columns`分区,两种都支持整数、字符串、日期。，不支持浮点数，不支持text、blob;

* columns分区**支持多列分区**

栗子：

```mysql
mysql> create table rc3(
    -> a int,
    -> b int)
    -> partition by range columns(a,b)(
    -> partition p01 values less than (0,10),
    -> partition p02 values less than (10,10),
    -> partition p03 values less than (10,20),
    -> partition p04 values less than (10,35),
    -> partition p05 values less than (10,maxvalue),
    -> partition p06 values less than (maxvalue,maxvalue)
    -> );
Query OK, 0 rows affected (0.25 sec)
```



解释一下：

元组的比较，(a,b) 、(c,d)比较

如果a>b，就不用比较b,d大小，否则，比较b,d;





## `HASH`分区

MySQL分为两种算法分区：

1. 常规Hash分区（取模）
2. 线性Hash分区（linear hash分区；线性的2的幂的运算法则）

```mysql
create tables xxx ()
partition by （linear） hash(expr) partitions num;
#expr 分区列，num分几个区
```

expr可以是MySQL中有效的任何函数或者其他表达式，只要他们返回一个既非常数，又非随机的整数，每次增加删除修改的时候都要计算一次，影响性能。

常规Hash分区和线性Hash分区区别

* 常规HASH在分区管理上很麻烦，如果要增加分区，之前的分区就要重新计算，
* 线性Hash分区维护的时候很方便，但是分区之间的数据分布不均匀，关于线性Hash分区算法自行看书，不难。

注意：Hash分区只支持整数分区。明白他们的算法不是取模就是取最小2的幂值



## `KEY`分区

和Hash分区leisure，但是不允许用户使用自定义的表达式。使用MySQL服务器提供的Hash函数，支持除Blob、Text类型之外其他类型的列作为分区键。

```mysql
create table xxx()
partition by key(expr) partitions num;
```

expr 可以是0个或多个字段名的字段。

默认是主键作为分区键，没有主键，默认使用唯一键（必须非空），



## 分区处理null方式

一般情况下分区把null当作空值，或者最小值处理。

* `range`分区中，`null`当最小值处理

* `list`分区中，`null`必须出现在枚举列表中

* `hash\key`中，当作0处理

解决办法：使用非空字段、默认值绕开MySQL默认NULL值得处理



# 分区的管理

`MySQL`中提供添加、删除、重定义、合并、拆分分区的命令，使用`alter table`实现。



## `range`&`list`分区管理

**删除**

同时也会删除对应分区内的数据

```mysql
alter table xxx drop partition partition_name;
```

**添加**

range添加分区需要从最大值侧添加，类似磁盘分区

list添加，不能出现重复内容

range为例

```mysql
alter table xxx add partition(partition partition_name value less than (xxx));
```

**重定义、合并、拆分分区**

**拆分**

```mysql
alter table xxx reorganize partition xxx into (partition p1 values xxx,partition p2 values xxx,)
```

**合并**

```mysql
alter table xxx reorganize partition p1,p2,p3 into (partition p1 values xxx)
```



**查看分区**

```mysql
mysql> explain partitions   select * from stu where id>40 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: stu
   partitions: p2
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
        Extra: Using where
1 row in set (0.00 sec)

ERROR:
No query specified
```





* list区间的添加，需要添加新的区间，然后重定义区间

* 重定义range分区的时候只能定义相邻的分区，重定义的分区区间必须和原分区区间覆盖相同的区间，不能改变区间类型。

* 重定义list分区的时候只能定义相邻的分区

  

* 单张表到单个文件：表的每一行都存储在同一个文件中。
* 张表到多个文件：用于非常大的表或要求数据在存储级别并行物理分离的表。
* 多张表到单个文件：用于小表(如查找表和代码表)。



## `hash`&`key`分区管理

添加、删除分区直接修改分区个数

```mysql
alter table xxx coalesce partition num;
```



# 使用分区的优点

1. 分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备。
2. 和单个磁盘或者文件系统相比，可以存储更多数据
3. 优化查询。在where语句中包含分区条件时，可以只扫描一个或多个分区表来提高查询效率；涉及sum和count语句时，也可以在多个分区上并行处理，最后汇总结果。
4. 分区表更容易维护。例如：想批量删除大量数据可以清除整个分区。
5. 可以使用分区表来避免某些特殊的瓶颈，例如InnoDB的单个索引的互斥访问。



# 分区的限制

1.一个表最多只能有1024个分区

2.如果分区字段中有主键或者唯一索引的列，那么多有主键列和唯一索引列都必须包含进来。即：分区字段要么不包含主键或者索引列，要么包含全部主键和索引列。

3.**分区表中无法使用外键约束**

4.MySQL的分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区，也不能只对索引分区而不对表分区，也不能只对表的一部分数据分区。



# 区别分区和分表

## 分表

分表是将一个大表按照一定的规则分解成多张具有独立存储空间的实体表，我们可以称为子表，每个表都对应三个文件，MYD数据文件，.MYI索引文件，.frm表结构文件。这些子表可以分布在同一块磁盘上，也可以在不同的机器上。

app读写的时候根据事先定义好的规则得到对应的子表名，然后去操作它。



## 分区

分区和分表相似，都是按照规则分解表。不同在于分表将大表分解为若干个独立的实体表，**而分区是将数据分段划分在多个位置存放**，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。

app读写的时候操作的还是大表名字，db自动去组织分区的数据。