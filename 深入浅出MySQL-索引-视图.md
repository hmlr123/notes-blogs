---
title: 深入浅出MySQL-索引&视图
date: 2019-04-06 20:30:25
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



索引的设计与使用、视图操作

核心内容：

视图的创建、<span style="color:red">联合索引</span>

<!--more-->

# 授人以渔

使用`explain`查看SQL语句执行情况，优化语句



# 索引



## 索引分类

在 MySQL 中，主要有四种类型的索引，分别为：`B-Tree `索引，`Hash` 索引，`Fulltext` 索引和` R-Tree` 索引。



### B-Tree 索引

`B-Tree` 索引是 MySQL 数据库中使用最为频繁的索引类型，使用树形结构



### Hash索引

散列是一种使用键值开启数据的快速直接访问的技术。 使用一种算法将键值转换成一个指针, 该指针指向包含这些键值的行的物理位置。存储地址=Hash(key） Hash-散列函数

缺点：

1. Hash 索引仅仅只能满足“=”,“IN”和“<=>”查询，不能使用范围查询；

2. Hash 索引无法被利用来避免数据的排序操作；

3. Hash 索引不能利用部分索引键查询；

4. Hash 索引在任何时候都不能避免表扫面；

5. Hash 索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高；

   

### 全文索引

- 目前来说，仅有`CHAR，VARCHAR`和`TEXT`这三种数据类型的列可以建` Full-text`索引。`MyISAM`支持。

- like+%只适合文本较少的情况

使用`match...against`进行全文索引`match `匹配  `against `标识关键字

```mysql
#使用match和against函数  
select * from mm_product where match(name,label) against('白猫 洗洁精'); 
```



### R-Tree索引

解决空间数据检索的问题，在 MySQL中，支持一种用来存放空间信息的数据类型GEOMETRY



## 索引与引擎

- MyISAM和InnoDB默认创建的是B-Tree索引

- MySQL支持全文本索引、前缀索引

- Memery默认使用Hash索引，也支持B-Tree索引

  

## 索引的增删改查

**创建表的时候直接指定索引**

```mysql
create table mytable(
id int not null,
name varchar(16) not null,
index [indexName](name(length))
);
```

栗子

```mysql
mysql> create table test3(
    -> id int not null,
    -> name varchar(15) not null,
    -> index ind_text3_name(name(10))
    -> );
Query OK, 0 rows affected (0.08 sec)

mysql> show create table test3 \G;
*************************** 1. row ***************************
       Table: test3
Create Table: CREATE TABLE `test3` (
  `id` int(11) NOT NULL,
  `name` varchar(15) NOT NULL,
  KEY `ind_text3_name` (`name`(10))
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

ERROR:
No query specified
```

**创建索引**：

```mysql
create [unique|fulltext|spatial] index index_name
[using index_type]
on table_name(index_col_name,...)

index_col_name:
col_name[(length)][asc|desc]
```

解释：

* `unique|fulltext|spatial`索引类型

* `using index_type`表示索引的具体实现方式，在MySQL中，有两种不同的索引：BTREE索引和HASH索引。

* `index_col_name`表示需要创建索引的字段名称，我们还可以针对多个字段创建复合索引，只需要在多个字段名称之间以英文逗号隔开即可。

* 如果是`char`或者`varchar`类型，`length`可任意小于字段实际长度，如果是blob或者text类型，必须指定长度

```mysql
create index cityname on city(city(10));
```

栗子:这个栗子是接着“创建表的时候直接指定索引”写的

```mysql
mysql> create index ind_test3_name_1 on test3(name(10));
Query OK, 0 rows affected (0.23 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table test3 \G;
*************************** 1. row ***************************
       Table: test3
Create Table: CREATE TABLE `test3` (
  `id` int(11) NOT NULL,
  `name` varchar(15) NOT NULL,
  KEY `ind_text3_name` (`name`(10)),
  KEY `ind_test3_name_1` (`name`(10))
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

ERROR:
No query specified
```

说明<span style="color:red">一个字段可以有多个索引</span>

**删除索引**

```mysql
drop index index_name on table_name
```

**修改表结构（添加索引）**

```mysql
alter table tablename add index indexName(columnName)
```

**查看表索引**

```mysql
mysql> show index from test3 \G;
*************************** 1. row ***************************
        Table: test3
   Non_unique: 1
     Key_name: ind_text3_name
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: 0
     Sub_part: 10
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 2. row ***************************
        Table: test3
   Non_unique: 1
     Key_name: ind_test3_name_1
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: 0
     Sub_part: 10
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
2 rows in set (0.00 sec)

ERROR:
No query specified
```

**查看索引使用情况**

```mysql
show status like 'handler_read%';
#hander_read_key值越高，越高表示索引查询到的次数，查询高效
#hander_read_rnd_next值越高，查询低效
```



**区分key和index**

key 是数据库的物理结构，它包含两层意义，一是约束（偏重于约束和规范数据库的结构完整性），二是索引（辅助查询用的）。包括primary key, unique key, foreign key 等。

index是数据库的物理结构，它只是辅助查询的，它创建时会在另外的表空间（mysql中的innodb表空间）以一个类似目录的结构存储。索引要分类的话，分为前缀索引、全文本索引等；因此，索引只是索引，它不会去约束索引的字段的行为。

## 索引利弊

利：

* 提高检索 

* 降低数据的排序成本

弊：

* 占用存储空间
* 资源消耗，增加更新带来的IO量和调整索引所导致的计算量



## 创建索引情况

1. 较频繁的作为查询条件的字段应该创建索引；
2. 唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件；
3. 更新非常频繁的字段不适合创建索引；



## 高性能索引策略 

优化篇在深入

1. 独立的列

2. 前缀索引对于内容很长的列，比如blob,text或者很长的varchar列，索引这些列的完整长度代价过高，可以考虑使用前缀索引。

3. 多列索引

4. **<span style="color:red">复合索引</span>**

   最左前缀匹配原则：mysql建立联合索引时会遵循最左前缀的原则，即最左优先。

   ![](https://i.loli.net/2019/04/06/5ca8ad40389c7.png)

   

   联合索引

   ![](https://i.loli.net/2019/04/07/5ca95217bcd3b.png)

    ![](https://i.loli.net/2019/04/07/5ca953ac87053.png)

5. 选择合适的索引顺序
   * 从左到右的顺序选择索引	
   * 考虑全局基数和选择性，解释一下，就是我们所说的选择性（`distinct values/all values`），全局基数就是`all values `,

## 存在索引但不使用索引的场景

1. 如果like 是以％开始；
2. 数据类型出现隐式转换，比如我们where value=2，表中的类型是varchar，那么value数据类型会转换成varchar。
3. 复合索引的情况下，查询条件不满足索引最左的原则，就是说索引要放在最左边
4. MySQL使用索引比全局扫描慢的情况
5. 用or分割开的条件，or前条件有索引，or后的列没有索引



# 视图

## what?

视图（view）是一种虚拟存在的表，是一个逻辑表，本身并不包含数据。作为一个select语句保存在数据字典中的。

通过视图，可以展现基表的部分数据；视图数据来自定义视图的查询中使用的表，使用视图动态生成。



现在大部分视图可以用工具实现，没必要写sql语句，sql语句有些麻烦

## 创建视图

```mysql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    VIEW view_name [(column_list)]
    AS select_statement
   [WITH [CASCADED | LOCAL] CHECK OPTION]
```

注意：视图在from关键字 后面不能包含子查询

解释：

`OR REPLACE`：或者替代已经存在的视图

`ALGORITHM`：选择那种视图算法

`select_statement`：select语句

`WITH [CASCADED | LOCAL] CHECK OPTION`：视图在修改更新时的权限

* `local`满足本视图条件就可以更新
* `cascade`满足所有视图条件才可以更新
* 默认是`cascade`



## 删除视图

可以一次删除多个视图

```mysql
DROP VIEW [IF EXISTS]   
view_name [, view_name] ...
```



## 修改视图

```mysql
ALTER
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = { user | CURRENT_USER }]
    [SQL SECURITY { DEFINER | INVOKER }]
VIEW view_name [(column_list)]
AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```

`DEFINER`：指出谁是视图的创建者或定义者

`SQL SECURITY`：要查询一个视图，首先必须要具有对视图的select权限。



## 查看视图

查看视图信息

```mysql
show table status like 'staff_list' \G
```

查询视图定义

```mysql
show create view xxx \G
```

有关视图的信息记录在information_schema数据库中的views表中

```mysql
select * from information_schema.views 
```







部分内容摘自老师的PPT

参考链接：

<https://tech.meituan.com/2014/06/30/mysql-index.html>

<https://www.cnblogs.com/geaozhang/p/6792369.html>