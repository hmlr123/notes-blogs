---
title: 深入浅出MySQL-存储引擎
date: 2019-04-06 12:40:33
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



常见MySQL存储引擎比较，及使用情况

<!--more-->

英语词汇：

restrict		限制

constraint	   约束

cascade	       串联

optimize             优化

# 查看数据库引擎

查看当前的默认存储引擎，可以使用如下的命令：

   ```mysql
 mysql > show variables like ‘table_type’;
   ```

查看`mysql`提供的引擎

```mysql
show engines \G;
```

```mysql
show variables like 'have%';
```

查看当前默认引擎

```mysql
show variables like '%storage_engine%';
```

查看某个表使用什么引擎

```mysql
show create table 表名
```



# 各种引擎特点

![](https://i.loli.net/2019/04/06/5ca8300abac34.png)



## MyISAM

**`MyISAM`不支持事务，也不支持外键，其优势是访问的速度快，对事务完整性没有要求或者以`SELECT、INSERT`为主的应用基本上都可以使用这个引擎来创建表；**

MyISAM磁盘上存储成三个文件，文件名和表名都一样，扩展名分别是：

* `.frm`（存储表定义）
* `.DYD `（MYdata 存储数据）
* `.MYI`（MYIndex 存储引擎）

数据文件和索引文件可以放置在不同的目录，需要在创建表的时候通过`DATA DIRECTORY`和`INDEX DIRECTORY`语句指定，也就是说不同的`MyISAM`表的索引文件和数据文件可以防止到不同的路径下。文件路径需要的是绝对路径，并且具有访问权限

MyISAM的表还支持3中不同的存储格式，分别是：

1. 静态（字段都是固定长度）表；

2. 动态表；

3. 压缩表；

静态表**默认存储方式，里面的字段是非变长字段，**这样每个记录都是固定长度，优点是**存储迅速，容易缓存，出现故障容易恢复；缺点占用空间大**。**数据在存储时，需要保存的内容后面用空格补充，返回值的时候，后面不带空格。**

动态表是**变长字段，频繁的更新删除记录会产生碎片**，需要定期执行`optimize table` 或者 `myisamchk-r`来改善性能。但是出现故障时恢复难

压缩表，每个记录都是单独压缩的



## InnoDB

`InnoDB`存储引擎提供了**具有提交、回滚和崩溃恢复能力的事务安全**。但是对比`MyISAM`的存储引擎，`InnoDB`**写的处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引**；

### 自动增长

* 自动增长字段如果为null 或者0，实际增长的值就是自动增长后的值
* 每次自动增长都会是从最大的那个值开始自增

* 使用`alter table *** auto_increment=n;`时语句强制设置成自动增长，默认从1开始，但是**该修改是保存在内存中的，如果数据重启，默认值会丢失，需要重新设置**
* `last_insert_id()`查询当前线程最后插入记录的值
* 如果一次插入多个值，返回的是第一条记录使用的自动增长
* **自动增长必须是索引，如果是组合索引，也必须是组合索引的第一列**，`MyISAM`没有这个限制

### 外键约束

创建外键

```mysql
mysql> create table city(
    -> city_id smallint unsigned not null auto_increment,
    -> city varchar(50) not null,
    -> country_id smallint unsigned not null,
    -> last_update timestamp not null default current_timestamp on update current_timestamp,
    -> primary key(city_id)
    -> key idx_fk_country_id (country_id),
    -> constraint 'fk_city_country' foreign key(country_id) references country (country_id) on delete restrict on update cascade
    -> )engine=innodb default charset=utf8;
```

* 父表必须要有对应的索引，子表在创建外键的时候也会自动创建对应夫人索引
* 在创建子表索引时，可以指定删除、修改父表时，字表的相应操作，包括`restrict`、`cascade`、`set null`、`no action`。`restrict`和`no action`作用一样，限制在子表有关联的情况下父表不能更新，`cascade`表父表在更新或删除的时候，更新或删除子表对应记录。`set null`表示父表在更新或删除的时候，子表对应字段被`set null`
* 当某个表被其他表创建外键参照的时候，该表的对应索引或者主键禁止被删除
* 导入数据的时候，可以关闭外键约束`set foreign_key_checks=0;`导入数据完成之后，使用`set foreign_key_checks=0;`恢复

### 存储方式

MySQL的Innodb包含两种表空间文件模式，默认的共享表空间和每个表分离的独立表空间。

**一般来说，当数据量很小的时候建议使用共享表空间的管理方式。数据量很大的时候建议使用独立表空间的管理方式。**

**共享表空间：** 

**Innodb的所有数据保存在一个单独的表空间里面**，而这个表空间可以由很多个文件组成，一个表可以跨多个文件存在，所以其大小限制不再是文件大小的限制，而是其自身的限制。

> 从Innodb的官方文档中可以看到，其表空间的最大限制为64TB，也就是说，Innodb的单表限制基本上也在64TB左右了，当然这个大小是包括这个表的所有索引等其他相关数据。

优点：

1. 以放表空间分成多个文件存放到各个磁盘上（表空间文件大小不受表大小的限制，如一个表可以分布在不同的文件上）。
2. 表数据和表描述放在一起方便管理。

缺点：所有的数据和索引存放到一个文件中，将有一个很庞大的文件，虽然可以把一个大文件分成多个小文件，但是多个表及索引在表空间中混合存储，这样对于一个表做了大量删除操作后表空间中将会有大量的空隙，特别是对于统计分析，日志系统这类应用最不适合用共享表空间

**独立表空间（在配置文件（my.cnf）中设置innodb_file_per_table=1）：**

独立表空间是每个表都有独立的多个数据文件，而且做到了索引和数据的分离，每一个表都有一个`.frm`表描述文件，还有一个`.ibd`文件(这个文件包括了单独一个表的数据内容以及索引内容)。

优点：

1. 每个表都有自已独立的表空间。

2. 每个表的数据和索引都会存在自已的表空间中。

3. 可以实现单表在不同的数据库中移动

4. 空间可以回收。

   > 对于使用独立表空间的表，不管怎么删除，表空间的碎片不会太严重的影响性能，而且还有机会处理（表空不能自已回收）,处理方式如下：
   > `Drop table`操作自动回收表空间
   >
   > 如果对于统计分析或是日志表，删除大量数据后可以通过:`alter table TableName engine=innodb;`回收不用的空间
   > 对于使`innodb-plugin`的`Innodb`使用`turncate table`也会使空间收缩

5. 使用独占表空间的效率以及性能会更高一点。

缺点：单表增加过大，如超过100个G

> 当使用独享表空间来存放Innodb的表的时候，每个表的数据以一个单独的文件来存放，这个时候的单表限制，又变成文件系统的大小限制了。

参考链接：<https://blog.csdn.net/nawenqiang/article/details/80010675>

## MEMORY

MEMORY存储引擎使用存在于内存中的内容来创建表，每个MEMORY表只实际对应一个磁盘文件，格式为`.frm`。MEMORY类型的表访问非常的快，因为它的数据值放在内存中的，并且**默认使用`HASH`索引**，但是一旦服务关闭，表中的数据就会丢失掉；

MEMORY表主要是用于那些内容变化不频繁的代码表，或者作为统计操作的中间结果表，便于高效地对中间结果进行分析并得到最终的统计结果。对存储引擎为MEMORY的表进行更新操作需要谨慎，因为数据并没有实际写入到磁盘中，所以一定要对下次重新启动服务之后如何获取这些修改后的数据所考虑；

## MERGE

MERGE存储引擎把一组MyISAM数据表当做一个逻辑单元来对待，让我们可以同时对他们进行查询。构成一个MERGE数据表结构的各成员MyISAM数据表必须具有完全一样的结构。每一个成员数据表的数据列必须按照同样的顺序定义同样的名字和类型，索引也必须按照同样的顺序和同样的方式定义。

常用于多表合并。查询时他会把所有表中的数据查询出来，插入数据时需要指定表

假设日志数据表的当前集合包括 log_2004、log_2005、log_2006、log_2007 ，而你可以创建一个如下所示的MERGE数据表把他们归拢为一个逻辑单元：

```mysql
CREATE TABLE log_YY  

(  

  dt  DATETIME NOT NULL,  

  info VARCHAR(100) NOT NULL,  

  INDEX (dt)  

) ENGINE = MyISAM;  
```

```mysql
CREATE TABLE log_merge  

(  

    dt DATETIME NOT NULL,  

    info VARCHAR(100) NOT NULL,  

    INDEX(dt)  

) ENGINE = MERGE UNION = (log_2004, log_2005, log_2006, log_2007);  
```





## TokuDB



第三方存储引擎

特性：

* Fractal树索引保证高效的插入性能
* 优秀的压缩性能，比InnoDB高近10倍
* 支持在线创建索引和田间删除属性列等DDL操作
* 使用Bulk Loader快速加载数据
* 主从延迟消除技术
* 支持ACID和MVCC

适用情况：

1. 日志数据，日志数据通常插入平凡，存储量大
2. 历史数据，通常不会有写数据，利用TokuDB的高压缩特性存储
3. 在线DDL较频繁的场景，提高可用性



# 引擎选择

![](https://i.loli.net/2019/04/06/5ca8520169b52.png)

注意：5.5以前默认引擎是MyISAM，5.5之后是InnoDB