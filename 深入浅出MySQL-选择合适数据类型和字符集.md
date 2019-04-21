---
title: 深入浅出MySQL-选择合适数据类型和字符集
date: 2019-04-06 16:33:32
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



合适的才是最好的！

<!--more-->

# 选择合适字符类型

## char与varchar

* char列数据在检索的时候会删除尾部的空格，对长度变化不大，并且对查询速度速度有较高要求的可以考虑使用char
* varchar列数据 ''表示一个字节
* char处理速度比varchar快

![](https://i.loli.net/2019/04/06/5ca865cb0bc1d.png)

InnoDB的char和varchar性能主要考虑行使用的存储总量



## text和blob

常见问题：

1. BLOB和TEXT值会引起一些性能问题，特别是在执行了大量的删除操作的时候；

   删除操作会在数据表中留下很大的空洞，以后填入这些空洞的记录性能上有影响；建议经常使用`optimize table`功能对表的碎片整理，避免空洞导致性能问题。

2. 可以使用合成的（`Synthetic`）索引来提高大文本字段（BLOB或者TEXT）的查询性能。

   合成索引就是根据大文本的内容建立一个散列值，并存储在单独的列中，这种只适用于精确的匹配查询，使用`md5()、sha1()、crc32()`或者自己的应用程序逻辑计算。如果散列值尾部有空格，不能存储在char或者varchar中。

3. 在不必要的时候避免检索大型的BLOB或者TEXT的值；

4. 把BLOB或者TEXT列分离到单独的表中；

   把原数据表中的数据列转换为固定长度的数据行数据，减少主表中的碎片，得到固定数据行的性能优势。



## 浮点数和定点数

1. 浮点数存在误差问题；

2. 对货币等精度敏感的数据，应该使用定点数表示或者存储；

3. 在编程中，如果用到浮点数，要特别注意误差问题，   并尽量避免做浮点数的比较；

4. 要注意浮点数中一些特殊值的处理；



## 日期类型

1. 根据需要选择能够满足应用的最小存储的日期类型；

2. TIMESTAMP表示的日期范围比DATETIME要短的多；

3. 如果记录的日期需要让不同的时区的用户使用，那么最好使用TIMESTAMP，因为日期类型中只有它能够和实际的时区对应；



# 字符集

查看可用字符集的命令

```mysql
mysql> show character set;
+----------+-----------------------------+---------------------+--------+
| Charset  | Description                 | Default collation   | Maxlen |
+----------+-----------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 |
| dec8     | DEC West European           | dec8_swedish_ci     |      1 |
| cp850    | DOS West European           | cp850_general_ci    |      1 |
| hp8      | HP West European            | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian       | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European        | latin1_swedish_ci   |      1 |
...
```

```mysql
mysql> desc information_schema.character_sets;
+----------------------+-------------+------+-----+---------+-------+
| Field                | Type        | Null | Key | Default | Extra |
+----------------------+-------------+------+-----+---------+-------+
| CHARACTER_SET_NAME   | varchar(32) | NO   |     |         |       |
| DEFAULT_COLLATE_NAME | varchar(32) | NO   |     |         |       |
| DESCRIPTION          | varchar(60) | NO   |     |         |       |
| MAXLEN               | bigint(3)   | NO   |     | 0       |       |
+----------------------+-------------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```



校对规则（collation）定义比较字符串的方式，字符集和校对规则是一对多关系

查看校对规则

```mysql
mysql> show collation like 'utf8%';
+--------------------------+---------+-----+---------+----------+---------+
| Collation                | Charset | Id  | Default | Compiled | Sortlen |
+--------------------------+---------+-----+---------+----------+---------+
| utf8_general_ci          | utf8    |  33 | Yes     | Yes      |       1 |
| utf8_bin                 | utf8    |  83 |         | Yes      |       1 |
| utf8_unicode_ci          | utf8    | 192 |         | Yes      |       8 |
```



校对规则命名：以相关的字符集名开始，通常包括一个语言名，以`_ci`（大小写不敏感）`_cs`(大小写敏感)、`_bin`(二元，基于字符编码的值比较)



## 字符集设置

### 服务器级字符集 校对规则设置

1. 在配置文件my.cnf中设置

2. 启动项中设置

   ```mysql
   C:\Users\liwei>mysqld character-set-server=utf8
   190406 17:31:00 [Note] --secure-file-priv is set to NULL. Operations related to importing and exporting data are disabled
   190406 17:31:00 [Note] mysqld (mysqld 5.5.62) starting as process 1920 ...
   ```



### 数据库级设置

先查看数据库默认字符集和校对规则

```mysql
show variables like 'character_set_database';
show variables like 'collation_database';
```

再创建数据库，创建数据库的时候指定字符集和校对规则

```mysql
mysql> create database test2 character set 'utf8' collate 'utf8_general_ci';
Query OK, 1 row affected (0.00 sec)

mysql> show create database test2;
+----------+----------------------------------------------------------------+
| Database | Create Database                                                |
+----------+----------------------------------------------------------------+
| test2    | CREATE DATABASE `test2` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+----------------------------------------------------------------+
1 row in set (0.00 sec)
```



### 数据表级设置

```mysql
create table zhazha(
id int(11) default null
)engine=innodb default charset=utf8 collate=utf_general_ci;
```



## 字符集的修改步骤

![](https://i.loli.net/2019/04/06/5ca875f4d0adc.jpg)