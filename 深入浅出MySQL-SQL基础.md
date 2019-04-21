---
title: 深入浅出MySQL-SQL基础
date: 2019-03-31 14:29:40
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---

从今天起，深入学习`MySQL` ，:smile:

<!--more-->



# 英语词汇

DDL(Data Definition Languages):数据定义语句，定义了不停的数据段、数据库、表、列、索引等数据库对象，常用关键字`create、drop、alter`等。

DML(Data Manipulation Languages):数据操纵语句，用于添加、删除、更新、查询数据库记录，并检查数据完整性，常用关键字`insert、delete、update、select`等。

DCL(Data Control Languages):数据控制语句，控制不同数据段直接的许可和访问级别，定义了数据库、表、字段、用户访问级别和安全级别。常用关键字`grant`、`revoke`等。



# 授人以鱼不如授人以渔

`？ contents`查看显示所有可供查询的分类，（我的数据库没安装这个，无法演示）

![](https://i.loli.net/2019/03/31/5ca06463aae97.jpg)



查看相关帮助：

例子

```mysql
mysql> ? create table
Name: 'CREATE TABLE'
Description:
Syntax:
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    [(create_definition,...)]
    [table_options]
    [partition_options]
    [IGNORE | REPLACE]
    [AS] query_expression

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    { LIKE old_tbl_name | (LIKE old_tbl_name) }
    ...
```



# DDL 数据定义语句

创建数据库

```mysql
create database test1;
```

使用数据库

```mysql
use test1;
```

删除数据库

```mysql
drop database test1;
```

创建表 `mysql`的表名以目录的形式存放于存盘中。

```mysql
mysql> create table emp(
    -> ename varchar(10) ,
    -> hiredate date ,
    -> sal decimal(10,2) ,
    -> deptno int(2)
    -> );
Query OK, 0 rows affected (0.09 sec)
```

<span style="color:red">小知识</span>:

`desc` 查看表结构 

```mysql
mysql> desc table_emp;
+----------+---------------+------+-----+---------+-------+
| Field    | Type          | Null | Key | Default | Extra |
+----------+---------------+------+-----+---------+-------+
| ename    | varchar(10)   | YES  |     | NULL    |       |
| hiredate | date          | YES  |     | NULL    |       |
| sal      | decimal(10,2) | YES  |     | NULL    |       |
| deptno   | int(2)        | YES  |     | NULL    |       |
+----------+---------------+------+-----+---------+-------+
4 rows in set (0.01 sec)

```

```mysql
#查看数据库创建详细信息
mysql> show create table table_emp \G;
*************************** 1. row ***************************
       Table: table_emp
Create Table: CREATE TABLE `table_emp` (
  `ename` varchar(10) DEFAULT NULL,
  `hiredate` date DEFAULT NULL,
  `sal` decimal(10,2) DEFAULT NULL,
  `deptno` int(2) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

ERROR:
No query specified
```

`\G`使记录能够按照字段竖向排列

删除表

```mysql
mysql> drop table table_emp;
```

修改表

* 修改表类型`modify`

  ```mysql
  alter table tablename modify [column] column_definition [first/after col_name]
  
  ```

  ```mysql
  mysql> alter table emp modify column ename varchar(10);
  Query OK, 0 rows affected (0.14 sec)
  Records: 0  Duplicates: 0  Warnings: 0
  ```

* 增加表字段`add`

  ```mysql
  alter table tablename add [column] column_definition [first/after col_name]
  ```

  ```mysql
  mysql> alter table emp add column age int(3);
  Query OK, 0 rows affected (0.19 sec)
  Records: 0  Duplicates: 0  Warnings: 0
  ```

* 删除表字段`drop`

  ```mysql
  alter table tablename drop [column] col_name
  ```

  ```mysql
  mysql> alter table emp drop qwe;
  Query OK, 0 rows affected (0.18 sec)
  Records: 0  Duplicates: 0  Warnings: 0
  ```

* 字段改名`change`

  ```mysql
  alter table tablename change [column] old_col_name_definition [first/after col_name]
  ```

  ```mysql
  mysql> alter table emp change age age1 int(4);
  Query OK, 0 rows affected (0.12 sec)
  Records: 0  Duplicates: 0  Warnings: 0
  ```

  `**change`和`modify`都可以修改表的定义，但是change后面需要写两次列名，可以修改列名，`modify`不能**

* 修改字段排列顺序

  注意前面有的`[first/after xx]`字段 

  ```mysql
  mysql> alter table emp add birth date after ename;
  Query OK, 0 rows affected (0.18 sec)
  Records: 0  Duplicates: 0  Warnings: 0
  ```

  其他类似

* 更改表名

  ```mysql
  alter table tablename rename [to] new_tablename;
  ```

  ```mysql
  mysql> alter table emp rename emp1;
  Query OK, 0 rows affected (0.07 sec)
  ```

  

# DML 数据操纵语句

1. 插入语句

```mysql
insert into tablename(field1,field2...,fieldn) values(value1,value2...valuen)
```



2. 修改（更新）语句

```mysql
update tablename set field1=value1,field2=value2...[where condition]
```



**可以同时更改多个表数据**

```mysql
update t1,t2,...set field1=xxx,field2=xxx... [where condition]
```

```mysql
mysql> update emp a,dept b set a.sal=a.sal*b.deptno,b.deptname=a.ename where a.deptno=b.deptno;
Query OK, 4 rows affected (0.09 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```

**多表更新常用于根据一个表的字段来动态修改另外一个表的的字段**



3. 删除记录

```mysql
delete from tablename [where condition]
```

**可以一次删除多个表记录**

```mysql
delete t1,t2,...tn from t1,t2,t3,...,tn [where condition]
```

```mysql
mysql> delete a,b from emp a, dept b where a.deptno=b.deptno;
Query OK, 3 rows affected (0.07 sec)
```

​	

 	4. 查询记录

```mysql
select * from tablename [where condition]
```

* `distinct `去掉重复的值，先查询，再去重复

```mysql
mysql> select distinct deptno from emp;
+--------+
| deptno |
+--------+
|      3 |
|      2 |
|      1 |
+--------+
3 rows in set (0.00 sec)
```

* 条件查询

```mysql
mysql> select * from emp where deptno in (select distinct deptno from emp );
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| tom   | 2003-12-02 | 200.00 |      3 |
| dony  | 2005-02-02 | 100.00 |      2 |
| tom   | 2003-12-02 | 200.00 |      1 |
| dony  | 2005-02-02 | 100.00 |      2 |
+-------+------------+--------+--------+
4 rows in set (0.00 sec)
```

* 排序和限制`order by`

```mysql
select * from tablename [where condition] [order by field1 [desc/asc],field2 [desc/asc],field3 [desc/asc]]...
```

`desc`降序，`asc`升序 默认升序 `order by`可以跟多个排序字段

`limit`限制显示的条数

```mysql
mysql> select * from emp order by deptno, sal desc limit 1,3;
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| dony  | 2005-02-02 | 100.00 |      2 |
| dony  | 2005-02-02 | 100.00 |      2 |
| tom   | 2003-12-02 | 200.00 |      3 |
+-------+------------+--------+--------+
3 rows in set (0.00 sec)
```

先按deptno排序，如果有字段相同，按desc排序，最后输出从第二条开始的3条记录

`limit`常用来分页

* 聚合

```mysql
select [field1,field2,field3,...fieldn] fun_name
from tablename
[where condition]
[group by field1,field2,field3,...fieldn
 [with rollup]]
[having where_condition]
```

`fun_name`:表示聚合函数，sum、count、max、max

`groud up`：表示要进行分类聚合的字段，比如按部门统计员工数

`with rollup`：可选字段，聚合之后的结果汇总

`having`：分类后的结果过滤

having 和 where区别：having是对聚合之后的结果过滤，where是聚合之前就对记录过滤。<span style="color:red">尽量先用`where`过来记录，这样结果集减少，聚合效率提高，最后在根据逻辑是否用`having`进行过滤</span>

```mysql
mysql> select deptno,count(1) from emp group by deptno;
+--------+----------+
| deptno | count(1) |
+--------+----------+
|      1 |        1 |
|      2 |        2 |
|      3 |        1 |
+--------+----------+
3 rows in set (0.00 sec)

mysql> select deptno,count(*) from emp group by deptno;
+--------+----------+
| deptno | count(*) |
+--------+----------+
|      1 |        1 |
|      2 |        2 |
|      3 |        1 |
+--------+----------+
3 rows in set (0.00 sec)

mysql> select deptno,count(1) from emp group by deptno with rollup;
+--------+----------+
| deptno | count(1) |
+--------+----------+
|      1 |        1 |
|      2 |        2 |
|      3 |        1 |
|   NULL |        4 |
+--------+----------+
4 rows in set (0.00 sec)

mysql> select deptno,count(1) from emp group by deptno having count(1)>1;
+--------+----------+
| deptno | count(1) |
+--------+----------+
|      2 |        2 |
+--------+----------+
1 row in set (0.00 sec)

mysql> select sum(sal),max(sal),min(sal) from emp;
+----------+----------+----------+
| sum(sal) | max(sal) | min(sal) |
+----------+----------+----------+
|   600.00 |   200.00 |   100.00 |
+----------+----------+----------+
1 row in set (0.00 sec)
```

* 表连接

  1. 内连接：会显示出其他不匹配的记录

     就是用where语句

  2. <span style="color:red">**外连接**</span>：会显示两张表中相互匹配的记录

     左外连接 ：左边表全部显示，右边不匹配的为空

     右外连接 ：右边表全部显示，左边不匹配的为空

     完全外连接 ：两边都会显示，两边不匹配的为空

     ```mysql
     mysql> select ename,deptname from emp left join dept on emp.deptno=dept.deptno;
     +-------+----------+
     | ename | deptname |
     +-------+----------+
     | tom   | NULL     |
     | dony  | NULL     |
     | tom   | tech     |
     | dony  | NULL     |
     +-------+----------+
     4 rows in set (0.00 sec)
     ```

     

  3. 交叉连接：交叉连接返回左表中的所有行，左表中的每一行与右表中的所有行组合。交叉连接也称作笛卡尔积，没有用条件

     ```mysql
     mysql> select * from emp,dept;
     +-------+------------+----------+--------+--------+----------+
     | ename | hiredate   | sal      | deptno | deptno | deptname |
     +-------+------------+----------+--------+--------+----------+
     | tom   | 2003-12-02 |   200.00 |      3 |      1 | tech     |
     | tom   | 2003-12-02 |   200.00 |      3 |      5 | fine     |
     | dony  | 2005-02-02 |   100.00 |      2 |      1 | tech     |
     | dony  | 2005-02-02 |   100.00 |      2 |      5 | fine     |
     | tom   | 2003-12-02 |   200.00 |      1 |      1 | tech     |
     | tom   | 2003-12-02 |   200.00 |      1 |      5 | fine     |
     | dony  | 2005-02-02 |   100.00 |      2 |      1 | tech     |
     | dony  | 2005-02-02 |   100.00 |      2 |      5 | fine     |
     | za    | 2005-02-03 | 30000.00 |      3 |      1 | tech     |
     | za    | 2005-02-03 | 30000.00 |      3 |      5 | fine     |
     +-------+------------+----------+--------+--------+----------+
     10 rows in set (0.00 sec)
     ```

* 子查询 in、not in 、 = 、！=、exists、not exists等

   ```mysql
  mysql> select * from emp where deptno in(select deptno from dept);
  +-------+------------+--------+--------+
  | ename | hiredate   | sal    | deptno |
  +-------+------------+--------+--------+
  | tom   | 2003-12-02 | 200.00 |      1 |
  +-------+------------+--------+--------+
  1 row in set (0.00 sec)
   ```

* **记录联合**

  ```mysql
  select * from t1
  union|union all
  select * from t2
  ...
  union|union all
  select * from tn;
  ```

  `union`和`union all`区别`union all`把结果集直接合并到一起，而`union`将`union all`的结果进行一次`distinct`，去出重复的结果。

```mysql
mysql> select deptno from emp
    -> union all
    -> select deptno from dept;
+--------+
| deptno |
+--------+
|      3 |
|      2 |
|      1 |
|      2 |
|      1 |
|      5 |
+--------+
6 rows in set (0.00 sec)

mysql> select deptno from emp
    -> union
    -> select deptno from dept;
+--------+
| deptno |
+--------+
|      3 |
|      2 |
|      1 |
|      5 |
+--------+
4 rows in set (0.00 sec)
```



# DCL 数据控制语句 

DBA管理系统中的对象权限时使用。 了解

授予权限`grant`

```mysql
mysql> grant select,insert on test1.* to 'z1'@'localhost' identified by '123';
Query OK, 0 rows affected (0.06 sec)
```

移除权限`revoke`

```mysql
mysql> revoke insert on test1.* to 'z1'@'localhost';
Query OK, 0 rows affected (0.06 sec)
```



# 查询元数据

三种方式

1）show语句

2）从INFORMATION_SCHEMA数据库里查询相关表

3）命令行程序，如mysqlshow, mysqldump

## show语句

```mysql
show databases;  --列出所有数据库
show create database db_name;  --查看数据库的DDL
show tables; --列出默认数据库的所有表
show tables from db_name;  --列出指定数据库的所有表
show table status;  --查看表的描述性信息
show table status from db_name;
show create table tbl_name;  --查看表的DDL
show columns from tbl_name;  --查看列信息
show index from tbl_name;  --查看索引信息
```

有几种show语句还可以带有一条like 'pattern'字句，用来限制语句的输出范围，其中'pattern'允许包含'%'和'_'通配符，比如下面这条语句返回domaininfo表中以s开头的所有列:

```mysql
show columns from domaininfo like 's%';
```

```mysql
mysql> show create table emp;
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                       |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| emp   | CREATE TABLE `emp` (
  `ename` varchar(10) DEFAULT NULL,
  `hiredate` date DEFAULT NULL,
  `sal` decimal(10,2) DEFAULT NULL,
  `deptno` int(2) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```



剩下两种不做介绍，自行百度（渣渣威也不是很懂）:pensive:







