---
title: 深入浅出MySQL-SQL优化
date: 2019-04-21 09:18:17
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
top: true
---

重中之中

<!--more-->



# 英语词汇

好尴尬，貌似都认识。。。



# 如何分析SQL语句

## 分析SQL语句的方法

使用`show [session|global] status`获取服务器提供的信息，默认`session`

```mysql
mysql> show global status like 'Com%';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| Com_admin_commands        | 0     |
| Com_assign_to_keycache    | 0     |
| Com_alter_db              | 0     |
| Com_alter_db_upgrade      | 0     |
| Com_alter_event           | 0     |
| Com_alter_function        | 0     |
| Com_alter_procedure       | 0     |
| Com_alter_server          | 0     |
| Com_alter_table           | 6     |
| Com_alter_tablespace      | 0     |
```



使用`show processlist`查看当前MySQL进行的线程

```mysql
mysql> show processlist;
+----+------+----------------+------+---------+------+-------+------------------+
| Id | User | Host           | db   | Command | Time | State | Info             |
+----+------+----------------+------+---------+------+-------+------------------+
|  5 | root | localhost:6029 | NULL | Query   |    0 | NULL  | show processlist |
+----+------+----------------+------+---------+------+-------+------------------+
1 row in set (0.00 sec)
```



### `explain`

**使用explain、desc可以获取MySQL是如何执行select语句的信息，包括表连接和连接的顺序**

```mysql
mysql> explain select count(*) from actor \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: actor
         type: index
possible_keys: NULL
          key: idx_actor_last_name
      key_len: 137
          ref: NULL
         rows: 200
        Extra: Using index
1 row in set (0.00 sec)

ERROR:
No query specified
```

解释

`select_type`：表示select的类型，常见的取值simple（简单表，不使用表连接或子查询）、primary（主查询，外层的查询）、union（union语句中的第二个或者后面的查询语句）、subquery（子查询中的第一个select）等

`table`：输出结果集的表

<span style="color:red">`type`</span>：表示MySQL在表中找到所需行的方式，访问类型 重点理解

* `type=all` 全表扫描
* `type=index` 索引全扫描
* `type=range` 索引范围扫描,常用于<、<=、>、>=等操作符
* **`type=ref`** 使用非唯一索引或唯一索引的前缀索引，返回某个单独值得记录行，**个人理解就是范围结果不唯一，where条件是等于**

```mysql
mysql> explain select b.*, a.* from payment a, customer b where a.customer_id=b.customer_id \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: b
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 505
        Extra:
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: a
         type: ref
possible_keys: idx_fk_customer_id
          key: idx_fk_customer_id
      key_len: 2
          ref: sakila.b.customer_id
         rows: 13
        Extra:
2 rows in set (0.00 sec)

ERROR:
No query specified
```

* `type=eq_ref` 类似ref，区别在于使用唯一索引，每个索引键值，表中只有一条匹配记录，**多表连接**中使用`primary key` 或者 `unique key`作为关联条件 **就是说外键使用唯一不重复的，多表联合操作返回唯一值。**外表的一个元祖，内表只有唯一一条元组与之对应

* `type=const/system`，单表中最多有一个匹配行，查询起来非常迅速，所以这个匹配行中的其他列的值可以被优化器在当前查询中当作常量处理，根据主键`primary key` 或唯一索引 `unique index`进行查询。j

  **就是说查询条件的字段不重复唯一**就会当作常量处理。

  coast：查询的条件唯一

  system：coast的特例，表里面只有一条记录的时候

```mysql
mysql> explain select * from actor where actor_id=1 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: actor
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 2
          ref: const
         rows: 1
        Extra:
1 row in set (0.00 sec)

ERROR:
No query specified
```

* type=null，MySQL不用访问表或者索引，直接就能得到结果.

```mysql
mysql> explain select 1 from dual where 1 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
        Extra: No tables used
1 row in set (0.00 sec)

ERROR:
No query specified
```

* `type=REF_OR_NUL` 类似ref,只是索索条件包括，链接字段的值为空的情况
* `type=index_merge`索引合并优化
* `type=unique_subquery`：in后面是一个查询主键字段的子查询字段
* `type=index_subquery`：in后面是一个普通索引的子查询

`possible_keys`：查询的时候可能使用的索引

`key`：实际使用的索引

`key_len`：索引字段的长度

`rows`：扫描的行数

`extra`：执行的情况和描述

* `using where`表示优化器回表查询数据
* `using index`表示查询覆盖索引查询



使用`explain extended`查看优化器做的修改

使用`explain partition`查看分区情况

优化器会去掉恒成立的条件



### profile

**profile可以更清楚的了解SQL执行的过程**

查看`profiling`

```mysql
select @@profiling;
```

开启`profiling`

```mysql
set profiling=1;
```

使用`profile`

```mysql
mysql> show profiles;
+----------+------------+-------------------------------------+
| Query_ID | Duration   | Query                               |
+----------+------------+-------------------------------------+
|        1 | 0.00045525 | select count(* ) from actor         |
|        2 | 0.00022275 | expalinselect count(* ) from actor  |
|        3 | 0.00007975 | expalin select count(* ) from actor |
|        4 | 0.00019775 | explain select count(* ) from actor |
|        5 | 0.00031625 | explain select count(* ) from actor |
+----------+------------+-------------------------------------+
5 rows in set (0.00 sec)
```

使用`profile`查看CPU、磁盘IO等资源的消耗

```mysql
mysql> show profile cpu for query 5;
+----------------------+----------+----------+------------+
| Status               | Duration | CPU_user | CPU_system |
+----------------------+----------+----------+------------+
| starting             | 0.000096 | 0.000000 |   0.000000 |
| checking permissions | 0.000007 | 0.000000 |   0.000000 |
| Opening tables       | 0.000017 | 0.000000 |   0.000000 |
| System lock          | 0.000044 | 0.000000 |   0.000000 |
| init                 | 0.000010 | 0.000000 |   0.000000 |
| optimizing           | 0.000006 | 0.000000 |   0.000000 |
| statistics           | 0.000010 | 0.000000 |   0.000000 |
| preparing            | 0.000006 | 0.000000 |   0.000000 |
| executing            | 0.000011 | 0.000000 |   0.000000 |
| end                  | 0.000007 | 0.000000 |   0.000000 |
| query end            | 0.000015 | 0.000000 |   0.000000 |
| closing tables       | 0.000005 | 0.000000 |   0.000000 |
| freeing items        | 0.000079 | 0.000000 |   0.000000 |
| logging slow query   | 0.000002 | 0.000000 |   0.000000 |
| cleaning up          | 0.000003 | 0.000000 |   0.000000 |
+----------------------+----------+----------+------------+
15 rows in set (0.00 sec)
```

使用`show profile source for query xxx`可以查看SQL解析执行过程中每个步骤使用的源码文件、函数名、以及具体的源文件行数



注意事项：

**每次开启MySQL客户端都要开启`profiling`**



### `trace`

MySQL5.6提供了对SQL上网跟踪trace，了解优化器执行过程

使用流程

1. 开启trace 设置格式json ，设置trace最大能够使用的内存大小(避免默认内存导致显示不完整)

```mysql
set optimizer_trace='enabled=on' ,en_markers_in_json=on;
set optimizer_trace_max_mem_size=100000;
```

2. 执行想要trace的语句
3. 检查information_schema.optimizer_trace 知道SQL如何执行SQL

```mysql
select * from information_schema.optimizer_trace \G:
```



由于我的MySQL版本为5.5，不支持`trace`，所以不做例子





# 索引问题

索引分类见索引章节，大部分内容在该章节有所涉及

MySQL索引最常见的是采用`B-tree`（ALV平衡树），可以用于全关键字、关键字范围、关键字前缀查询。

## 使用索引的情况

1. 匹配全值
2. 匹配值的范围查询
3. 匹配最左前缀（复合索引中）
4. 仅仅对索引查询
5. 前缀索引
6. 能够实现索引匹配部分精确而其他部分进行范围查询

```mysql
mysql> explain select inventory_id from rental where rental_date='2006&2&14 15:16:03' and customer_id>=300 and customer_id<=400 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: rental
         type: ref
possible_keys: idx_fk_customer_id,idx_rental_date
          key: idx_rental_date
      key_len: 8
          ref: const
         rows: 182
        Extra: Using where; Using index
1 row in set (0.00 sec)

ERROR:
No query specified
```

解释：先用索引匹配出对应的数据地址，然后回表获取对应数据，在进行范围查询

7. 如果列名是索引，那么使用`column_name is null`会使用索引

```mysql
mysql> explain select * from payment where rental_id is null \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: payment
         type: ref
possible_keys: fk_payment_rental
          key: fk_payment_rental
      key_len: 5
          ref: const
         rows: 5
        Extra: Using where
1 row in set (0.10 sec)
```

8. 5.6引入`index Condition Pushdown(ICP)`特性优化查询，将某些条件过滤操作下放到存储引擎，以6为例，在5.6一下，数据库的操作如6的所说，但是在5.6及以后，数据库用索引查询出对应数据的地址后，不会立刻回表显示数据，而是在索引`customer_id`上过滤条件，再从表中读取数据，减少不必要数据的IO读操作



## 存在索引但是不能使用索引的情况

* %开头
* 数据类型出现隐式转换
* 复合索引不满足最左原则
* or分割

查看索引情况

```mysql
mysql> show status like 'Handler_read%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Handler_read_first    | 2     |
| Handler_read_key      | 1     |
| Handler_read_last     | 0     |
| Handler_read_next     | 202   |
| Handler_read_prev     | 0     |
| Handler_read_rnd      | 0     |
| Handler_read_rnd_next | 27    |
+-----------------------+-------+
7 rows in set (0.00 sec)
```

`Handler_read_key `低说明索引不好，用得少

`Handler_read_rnd_next`高说明查询低效，需要建立索引补救



# SQL优化步骤

单独写个章节



# 优化表方法

## 定期分析表

分析表 分析和存储表的关键字，得到准确的统计信息

```mysql
mysql> analyze table payment;
+----------------+---------+----------+----------+
| Table          | Op      | Msg_type | Msg_text |
+----------------+---------+----------+----------+
| sakila.payment | analyze | status   | OK       |
+----------------+---------+----------+----------+
1 row in set (0.09 sec)
```

检查表 检查单表夺标是否存在错误

```mysql
mysql> check table payment_myisam;
+-----------------------+-------+----------+----------+
| Table                 | Op    | Msg_type | Msg_text |
+-----------------------+-------+----------+----------+
| sakila.payment_myisam | check | status   | OK       |
+-----------------------+-------+----------+----------+
1 row in set (0.01 sec)
```



## 定期优化表

```mysql
mysql> optimize table payment_myisam;
+-----------------------+----------+----------+-----------------------------+
| Table                 | Op       | Msg_type | Msg_text                    |
+-----------------------+----------+----------+-----------------------------+
| sakila.payment_myisam | optimize | status   | Table is already up to date |
+-----------------------+----------+----------+-----------------------------+
```

会给出相应的优化信息



**注意**：以上操作都会锁定表



# 常用SQL语句优化

## 大批量插入数据

**对MyISAM存储引擎的表**

```mysql
alter table xxx disable keys;	#关闭MyISAM表非唯一索引的更新。
load the data;
alter table xxx enable keys;	#开启
```

```mysql
load data infile '路径/文件名' into table xxx;
```



**对InnoDB的表**

1. **InnoDB表的按照主键的顺序存储**，按主键的顺序存储数据效率高
2. 关闭唯一性校验，在存在索引中遇到。

```mysql
set unique_checks=0;
set unique_checks=1;
```

3. 关闭自动提交，然后在开启自动提交

```mysql
set autocommit=0;
set autocommit=1;
```



## 优化insert语句

* 一次插入多条数据
* 索引文件和数据文件放在不同的磁盘
* 用文本文件导入数据 `load data infile xxx into table xxx`
* 使用内存表



## 优化Order By语句

这部分看不懂，等老师讲

## 优化Group by语句

默认情况下`group by`会对分类的字段排序

避免排序结果的消耗，指定`order by null`禁止排序

```mysql
mysql> explain select payment_date,sum(amount) from payment group by payment_date \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: payment
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 16451
        Extra: Using temporary; Using filesort
1 row in set (0.00 sec)

ERROR:
No query specified
```

```mysql
mysql> explain select payment_date,sum(amount) from payment group by payment_date order by null\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: payment
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 16451
        Extra: Using temporary
1 row in set (0.00 sec)

ERROR:
No query specified
```

查看时间

```mysql
mysql> show profiles;
+----------+------------+------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                    |
+----------+------------+------------------------------------------------------------------------------------------+           |
|       18 | 0.00028650 | explain select payment_date,sum(amount) from payment group by payment_date               |
|       19 | 0.00015250 | explain select payment_date,sum(amount) from payment group by payment_date order by      |
|       20 | 0.00026600 | explain select payment_date,sum(amount) from payment group by payment_date order by null |
+----------+------------+------------------------------------------------------------------------------------------+
15 rows in set (0.00 sec)
```

可以看到query 18明显比20慢点儿，分析上面的explain，发现没有禁止排序分分类存在`using filesort`，而`filesort`消耗时间长



## 优化嵌套查询

子查询可以一次性完成多种逻辑操作，同时避免事务、或者表的锁死，但是有的情况下，子查询可以用`join`替代。原因在于**MySQL不需要创建临时表完成逻辑上需要两个步骤的查询工作**

```mysql
mysql> explain select * from customer where customer_id not in (select customer_id from payment)\G;
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: customer
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 577
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: payment
         type: index_subquery
possible_keys: idx_fk_customer_id
          key: idx_fk_customer_id
      key_len: 2
          ref: func
         rows: 13
        Extra: Using index
2 rows in set (0.00 sec)

ERROR:
No query specified
```

```mysql
mysql> explain select * from customer a left join payment b on a.customer_id=b.customer_id where b.customer_id is null \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: a
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 577
        Extra:
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: b
         type: ref
possible_keys: idx_fk_customer_id
          key: idx_fk_customer_id
      key_len: 2
          ref: sakila.a.customer_id
         rows: 13
        Extra: Using where; Not exists
2 rows in set (0.00 sec)

ERROR:
No query specified
```

profile分析

```mysql
*************************** 14. row ***************************
Query_ID: 22
Duration: 0.00227575
   Query: explain select * from customer where customer_id not in (select customer_id from payment)
*************************** 15. row ***************************
Query_ID: 23
Duration: 0.00043550
   Query: explain select * from customer a left join payment b on a.customer_id=b.customer_id where b.customer_id is null
15 rows in set (0.00 sec)

ERROR:
No query specified
```

可以看出第二种明显快很多

再看各部分消耗时间

```mysql
mysql> show profile for query 22;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000077 |
| checking permissions | 0.000007 |
| checking permissions | 0.000006 |
| Opening tables       | 0.001802 |#
| System lock          | 0.000032 |
| init                 | 0.000096 |
| optimizing           | 0.000010 |
| statistics           | 0.000029 |
| preparing            | 0.000012 |
| executing            | 0.000016 |
| optimizing           | 0.000006 |
| statistics           | 0.000010 |
| preparing            | 0.000023 |
| executing            | 0.000013 |
| end                  | 0.000010 |
| query end            | 0.000016 |
| closing tables       | 0.000012 |
| freeing items        | 0.000094 |
| logging slow query   | 0.000004 |
| cleaning up          | 0.000006 |
+----------------------+----------+
20 rows in set (0.00 sec)

mysql> show profile for query 23;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000094 |
| checking permissions | 0.000007 |
| checking permissions | 0.000007 |
| Opening tables       | 0.000036 |
| System lock          | 0.000037 |
| init                 | 0.000042 |
| optimizing           | 0.000014 |
| statistics           | 0.000042 |
| preparing            | 0.000020 |
| executing            | 0.000025 |
| end                  | 0.000009 |
| query end            | 0.000011 |
| closing tables       | 0.000010 |
| freeing items        | 0.000075 |
| logging slow query   | 0.000003 |
| cleaning up          | 0.000005 |
+----------------------+----------+
16 rows in set (0.00 sec)
```

可以看出第二种方式比第一种慢的原因在于`opending tables`,为什么是`opending tables`,是由于创建了第一种临时表，打开表消耗了时间?

由于我的版本不支持`trace`，就没办法分析优化器处理流程了



## MySQL如何优化or条件

含有or的每个条件必须是索引，才会使用索引，复合索引使用or不能使用索引



## 优化分页查询

正常情况下，我们使用limit查询时，会把前面的数据查询到了，但是有舍弃，造成不必要的数据去读操作，解决办法如下。

第一种，在索引上完成排序分页的操作，在根据索引回表获取其他数据。

第二种，添加一个字段，记录上一页的最后一行的标号，然后在查询的时候添加该字段的限制，准确的地址查询，避免，不必要的查询。但是这种情况是适用于不重复的情况，重复的情况下会出现分页结果内容丢失。



过程自行看书，网上关于分页的优化有很多，比树上强一些，google一下。



## SQL提示

`use index`希望使用提供的索引

```mysql
explain select * from table_name use index(xxx) \G;
```

`ignore index`忽略索引

```mysql
explain seletc * from table_name ignore index(xxx) \G;
```

`force index`强制使用某种索引

```mysql
explain select * from table_name force index(xxx) \G;
```



# 常用SQL技巧

## 正则表达式

使用`regexp`

![](https://i.loli.net/2019/04/21/5cbc3ffc058b7.jpg)



## 数据库名表名问题

MySQL的命名与操作系统有关，linux系统对大小写敏感，windows对大小写不敏感，建议命名的时候（大小写不敏感）不要相同。