---
title: 深入浅出MYSQL-事务控制和锁定语句
date: 2019-04-11 15:00:03
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---

对比java、spring的锁、事务机制思考学习

<!--more-->



# 存储类型与锁

![](<https://i.loli.net/2019/04/06/5ca8300abac34.png>)



# 授之以渔

锁：<https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html>

事务：<https://dev.mysql.com/doc/refman/5.7/en/commit.html>



# 锁

类似java的锁机制，java中可以用Lock锁住资源，如果当前锁被其他线程获取，该线程将进入等待队列，直到使用当前锁的线程释放当前锁。

mysql使用`lock tables`可以锁定当前线程的表，当表被锁定，当前线程就会等待，直到获取锁为止。

使用`unlock tables`可以释放当前线程获得任意锁，当前线程执行另一个`lick tables`时，或当前与服务器的连接被关闭的时候，所有的锁会隐式解锁。

```mysql
LOCK TABLES
    tbl_name [[AS] alias] lock_type
    [, tbl_name [[AS] alias] lock_type] ...

lock_type: {
    READ [LOCAL]
  | [LOW_PRIORITY] WRITE
}

UNLOCK TABLES
```

* 可以看出一次可以锁定多个表

* `lock_type`  <span style="color:red">重点</span>

  查询是读，修改、增加、删除是写。

  write：写锁，当前线程使用写锁时，当前线程读写，即可以CRUD操作；其他线程读写操作会等待，直到当前持有锁的线程释放锁。

  read：读锁，当前线程使用读锁时，当前线程可以读内容，但是不能写，即只能读。其他线程可以读写，但是写操作会等待，直到占用锁的线程释放锁。

  

栗子：

使用表：

```mysql
mysql> select * from emp;
+-------+------------+----------+--------+
| ename | hiredate   | sal      | deptno |
+-------+------------+----------+--------+
| tom   | 2003-12-02 |   200.00 |      3 |
| dony  | 2005-02-02 |   100.00 |      2 |
| tom   | 2003-12-02 |   200.00 |      1 |
| dony  | 2005-02-02 |   100.00 |      2 |
| za    | 2005-02-03 | 30000.00 |      3 |
+-------+------------+----------+--------+
5 rows in set (0.07 sec)
```

比较说明，A：当前线程，B：其他线程



**读锁**

第一步：

A: 给当前线程的表emp加读锁

```mysql
mysql> lock table emp read;
Query OK, 0 rows affected (0.00 sec)
```

B:无操作



第二步：

A:查询

```mysql
mysql> select * from emp where deptno=1;
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| tom   | 2003-12-02 | 200.00 |      1 |
+-------+------------+--------+--------+
1 row in set (0.00 sec)
```

B:查询

```mysql
mysql> select * from emp where deptno=1;
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| tom   | 2003-12-02 | 200.00 |      1 |
+-------+------------+--------+--------+
1 row in set (0.00 sec)
```

**说明读锁的情况下，其他线程也可以读**



第三步：

A：给当前表插入数据

```mysql
mysql> insert into emp values('hello','1002%1%1',246.666,6);
ERROR 1099 (HY000): Table 'emp' was locked with a READ lock and can't be updated
```

**说明读锁，当前线程不允许写操作**

B：给当前表插入数据

```mysql
mysql> insert into emp values('hello','1002%1%1',246.666,6);
```

进入等待状态；

**说明其他线程可以读写，但是写操作会等待**



第四步：

A:释放锁

```mysql
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

B：可以进行写操作

```mysql
Query OK, 1 row affected, 1 warning (2 min 41.82 sec)
```

**说明占用锁的线程释放锁后，其他线程可以写操作**



**写锁**

第一步：

A:加写锁

```mysql
mysql> lock tables emp write;
Query OK, 0 rows affected (0.00 sec)
```

B:无任何操作



第二步：

A:查询

```mysql
mysql> select * from emp where deptno=1;
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| tom   | 2003-12-02 | 200.00 |      1 |
+-------+------------+--------+--------+
1 row in set (0.00 sec)
```

B:查询

```mysql
select * from emp where deptno=1;
```

其他线程读等待。

说明，当前线程持有锁的时候可以读，其他线程读操作会等待。



第三步：

A:释放锁

```mysql
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

B:无任何操作

```mysql
+-------+------------+--------+--------+
| ename | hiredate   | sal    | deptno |
+-------+------------+--------+--------+
| tom   | 2003-12-02 | 200.00 |      1 |
+-------+------------+--------+--------+
1 row in set (1 min 33.69 sec)
```

说明当前线程占用写锁的时候，其他线程的读操作会等待，直到当前锁被释放，其他线程可以读。



第四步：

A:写操作

```mysql
mysql> insert into emp values('hello','1002%1%1',246.666,6);
Query OK, 1 row affected, 1 warning (0.07 sec)
```

B：写操作

```mysql
mysql> insert into emp values('hello','1002%1%1',246.666,6);
```

说明，当前线程持有锁的时候可以读，其他线程写操作会等待。



第五步：

A:释放锁

```mysql
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

B:无任何操作

```mysql
mysql> insert into emp values('hello','1002%1%1',246.666,6);
Query OK, 1 row affected, 1 warning (2 min 43.79 sec)
```

说明当前线程占用写锁的时候，其他线程的写操作会等待，直到当前锁被释放，其他线程可以写。



# 事务控制

通过set autocommit、start transaction、commit、rollback等语句支持本地事务。

```mysql
START TRANSACTION
    [transaction_characteristic [, transaction_characteristic] ...]

transaction_characteristic: {
    WITH CONSISTENT SNAPSHOT
  | READ WRITE
  | READ ONLY
}

BEGIN [WORK]
COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
SET autocommit = {0 | 1}
```

默认情况下，mysql是`autocommit`的，如果需要通过明确的`commit`和`rollback`来提交和回滚事务，那么需要通过明确的事务控制命令来开始事务

* `start transaction`或`begin`语句可以开始一项新的事务。
* `commit`和`rollback`用来提交或者回滚事务。
* `chain`和`release`子句分别用来定义在事务提交或者回滚之后的操作，`chain`会立即启动一个新事物，并且和刚才的事务具有相同的隔离级别，`release`则会断开和客户端的连接。
* SET AUTOCOMMIT可以修改当前连接的提交方式，如果设置了`set autocommit=0`，则设置
  之后的所有事务都需要通过明确的命令进行提交或者回滚。



## 事务例子

栗子：

使用的表：

```mysql
mysql> desc emp;
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

空表

A：当前线程

B：其他线程



A:开启事务

```mysql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
```

A:插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,1);
Query OK, 1 row affected (0.00 sec)
```

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

B:插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,2);
Query OK, 1 row affected (0.07 sec)
```

说明可以插入数据

B:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      2 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

可以看出A线程插入的数据没有显示,B插入的数据显示了

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

发现B线程插入的数据没有显示？？？这是什么鬼？？？

A:提交事务

```mysql
mysql> commit;
Query OK, 0 rows affected (0.07 sec)
```

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
| hello | 1234-12-04 | 12.12 |      2 |
+-------+------------+-------+--------+
2 rows in set (0.00 sec)
```

B：查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
| hello | 1234-12-04 | 12.12 |      2 |
+-------+------------+-------+--------+
2 rows in set (0.00 sec)
```



到这里是不是发现一些奇怪的问题，为什么A开启事务后没办法查询到B对表的操作，B也没办法查询到A的操作？？？

类比java多线程的内存模型就知道，java的内存模型存在线程内存和主内存。MySQL也是一样的，我们对一个线程开启事务，然后对他进行的操作是在操作系统的内存中进行的，而没开启事务则是在硬盘的存储空间中（这点不一样，java是在主存中，mysql是在硬盘的存储中，其实也一样，可以把mysql的硬盘存储理解成java中的主内存）。

那么现在问题迎刃而解了，事务的处理是在内存中的，所有读写操作都在内存中，而非事务的处理是在硬盘的存储中。两种情况针对的数据不同，自然读写出来的结果不一样了。

当事务提交后，当前线程在内存中读写数据就会写入到硬盘的存储中，其他其他县城也就可以访问该数据了，当前线程也可以访问表的其他数据。



## lock是否可以回滚

栗子二：

使用相同的表，数据为空

A:加写锁

```mysql
mysql> lock tables emp write;
Query OK, 0 rows affected (0.00 sec)
```

A：插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,1);
Query OK, 1 row affected (0.07 sec)
```

B:查询数据

```mysql
mysql> select * from emp;
```

进入等待

A：回滚记录

```mysql
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

B:还是等待状态

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

A:提交解锁

```mysql
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

B:等待结束

```mysql
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (1 min 20.08 sec)
```

**说明对`lock`不能回滚，回滚无效**

为什么事务可以回滚，锁不能呢？？

个人认为：锁是作用在硬盘的存储上，写操作就会将内容写进磁盘，没有回滚的余地；事务是作用在内存中，有回滚的余地。



## 锁情况下开启事务

> 如果在锁表期间，用`start transaction`开启了一个新事务，会造成一个隐式的`unlock tables`被执行

栗子三：验证锁情况下开启事务

A：写锁

```mysql
mysql> lock tables emp write;
Query OK, 0 rows affected (0.00 sec)
```

B:查询 进入等待状态

```mysql
mysql> select * from emp;
```

A:插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,1);
Query OK, 1 row affected (0.07 sec)
```

B:等待状态

A:回滚

```mysql
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

B:等待状态

A：开启事务

```mysql
mysql> start transaction
    -> ;
Query OK, 0 rows affected (0.00 sec)
```

B：等待解除

```mysql
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (1 min 46.22 sec)
```

可以看到开启新事务后，lock就被解除了。

> 因此，在同一个事务中，最好不使用不同存储引擎的表，否则rollback时需要对非事务
> 类型的表进行特别的处理，因为commit、rollback只能对事务类型的表进行提交和回滚。
>
> 通常情况下，只对提交的事务纪录到二进制的日志中，但是如果一个事务中包含非事务
> 类型的表，那么回滚操作也会被记录到二进制日志中，以确保非事务类型表的更新可以被复
> 制到从的数据库中。

那么为什么开启新事物锁就被解除了呢？？？

个人理解：开启新事物就意味着该线程可以读写该表内容，可以理解成事务的优先级比锁高。



## `savepoint`回滚

> 在事务中可以通过定义`savepoint`，指定回滚事务的一个部分，但是不能指定提交事务
> 的一个部分。对于复杂的应用，可以定义多个不同的`savepoint`，满足不同的条件时，回滚
> 不同的`savepoint`。需要注意的是，如果定义了相同名字的`savepoint`，则后面定义的
> savepoint会覆盖之前的定义。对于不再需要使用的`savepoint`，可以通过`release savepoint`
> 命令删除`savepoint`，删除后的`savepoint`，不能再执行`rollback to savepoint`命令。



栗子四：

A:开启事务

```mysql
mysql> start transaction;
Query OK, 0 rows affected (0.07 sec)
```

A:插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,1);
Query OK, 1 row affected (0.00 sec)
```

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

B:查询数据

```mysql
mysql> select * from emp;
Empty set (0.00 sec)
```

A:启用`savepoint`指定部分回滚内容

```mysql
mysql> savepoint test;
Query OK, 0 rows affected (0.00 sec)
```

A:插入数据

```mysql
mysql> insert into emp values('hello','1234*12(4',12.12,2);
Query OK, 1 row affected (0.00 sec)
```

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
| hello | 1234-12-04 | 12.12 |      2 |
+-------+------------+-------+--------+
2 rows in set (0.00 sec)
```

A:回滚部分数据

```mysql
mysql> rollback to savepoint test;
Query OK, 0 rows affected (0.00 sec)
```

A:提交事务

```mysql
mysql> commit
    -> ;
Query OK, 0 rows affected (0.06 sec)
```

B:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```

A:查询数据

```mysql
mysql> select * from emp;
+-------+------------+-------+--------+
| ename | hiredate   | sal   | deptno |
+-------+------------+-------+--------+
| hello | 1234-12-04 | 12.12 |      1 |
+-------+------------+-------+--------+
1 row in set (0.00 sec)
```





# 分布式事务

~~用的少，不做介绍，自己看书去，mmp，手都酸了。~~

目前阶段我还用不上，但是了解下相关概念，为以后打下基础。

在开发中，为了降低单点压力，通常会根据业务情况进行分表分库，将表分布在不同的库中（库可能分布在不同的机器上）。在这种场景下，事务的提交会变得相对复杂，因为多个节点（库）的存在，可能存在部分节点提交失败的情况，即事务的ACID特性需要在各个不同的数据库实例中保证。比如更新db1库的A表时，必须同步更新db2库的B表，两个更新形成一个事务，要么都成功，要么都失败。



## 分布式服务器用途 

了解下分布式服务器的用途：<https://juejin.im/entry/589abc01128fe10058fc542c> 



## 分布式事务原理

当前分布式事务只支持InnoDB存储引擎

在MySQL中，使用分布式事务的应用程序涉及到一个或多个资源管理器和一个事务管理器。

* **资源管理器（resource manager）**：用来管理系统资源，是通向事务资源的途径。数据库就是一种资源管理器。资源管理还应该具有管理事务提交或回滚的能力。

* **事务管理器（transaction manager）：**事务管理器是分布式事务的核心管理者。事务管理器与每个资源管理器（resource manager）进行通信，协调并完成事务的处理。事务的各个分支由唯一命名进行标识。

  

**mysql在执行分布式事务（外部XA）的时候，mysql服务器相当于xa事务资源管理器，与mysql链接的客户端相当于事务管理器。**



分布式事务分为两个阶段：

**阶段一为准备（prepare）阶段**。即所有的参与者准备执行事务并锁住需要的资源。参与者ready时，向transaction manager报告已准备就绪。 

**阶段二为提交阶段（commit）**。当transaction manager确认所有参与者都ready后，向所有参与者发送commit命令。 
如下图所示： 

![](https://img-blog.csdn.net/20170425103341298?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc29vbmZseQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 事务协调者transaction manager

因为XA 事务是基于两阶段提交协议的，所以需要有一个事务协调者（transaction manager）来保证所有的事务参与者都完成了准备工作(第一阶段)。如果事务协调者（transaction manager）收到所有参与者都准备好的消息，就会通知所有的事务都可以提交了（第二阶段）。MySQL 在这个XA事务中扮演的是参与者的角色，而不是事务协调者（transaction manager）。



## Mysql的XA事务分为外部XA和内部XA

* 外部XA用于跨多MySQL实例的分布式事务，需要应用层作为协调者，通俗的说就是比如我们在PHP中写代码，那么PHP书写的逻辑就是协调者。应用层负责决定提交还是回滚，崩溃时的悬挂事务。MySQL数据库外部XA可以用在分布式数据库代理层，实现对MySQL数据库的分布式事务支持，例如开源的代理工具：网易的DDB，淘宝的TDDL等等。

* 内部XA事务用于同一实例下跨多引擎事务，由Binlog作为协调者，比如在一个存储引擎提交时，需要将提交信息写入二进制日志，这就是一个分布式内部XA事务，只不过二进制日志的参与者是MySQL本身。Binlog作为内部XA的协调者，在binlog中出现的内部xid，在crash recover时，由binlog负责提交。(这是因为，binlog不进行prepare，只进行commit，因此在binlog中出现的内部xid，一定能够保证其在底层各存储引擎中已经完成prepare)。

以上内容摘自：<https://blog.csdn.net/soonfly/article/details/70677138>



## 语法

```mysql
XA {START|BEGIN} xid [JOIN|RESUME] 
#启动xid事务 (xid 必须是一个唯一值; 不支持[JOIN|RESUME]子句) 
XA END xid [SUSPEND [FOR MIGRATE]] 
#结束xid事务 ( 不支持[SUSPEND [FOR MIGRATE]] 子句) 
XA PREPARE xid 
#准备、预提交xid事务 
XA COMMIT xid [ONE PHASE] 
#提交xid事务 
XA ROLLBACK xid 
#回滚xid事务 
XA recover
#返回当前数据库中处于prepare状态的分支事务的详细信息
```

xid:事务标识符，用来标识一个分布式事务，一般有客户端会MySQL服务器生成，包括三部分

```mysql
xid: gtrid [, bqual [, formatID ]]
```

gtrid:分布式事务标识符

bqual:分支限定符，默认空串。

formatID:数字，标识gtrid和bqual值，默认1。



例子就不上了，自行百度。



# 少用他，性能低，又麻烦，还不安全，不完善

XA的性能很低。一个数据库的事务和多个数据库间的XA事务性能对比可发现，性能差10倍左右。因此要尽量避免XA事务，例如可以将数据写入本地，用高性能的消息系统分发数据。或使用数据库复制等技术。只有在这些都无法实现，且性能不是瓶颈时才应该使用XA。