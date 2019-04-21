---
title: 深入浅出MySQL-存储过程和函数
date: 2019-04-08 10:50:19
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---

重点是存储过程、函数，他们的书写格式，以及控制流程

有些作用，多练习！

<!--more-->



# 存储过程、函数用来干什么？

个人意愿是从事java开发，但是java开发无法避免涉及到数据库，很多时候我们需要一些测试数据，我们不可能一条一条的输入，这时候我们需要生成测试数据（可以使用第三方的软件实现），存储过程函数就开始起作用了。当然，在一些公司，比如金融、企业、政府等等，使用存储过程很广泛，存储过程一旦调试完成通过后就能稳定运行，这与各个业务在一段时间内是相对稳定和确定是匹配的；存储过程大大地减少了业务系统与数据库的交互，一定程度降低了业务系统与数据库的耦合。但是其可移植性差，很多情况下使用存储过程是得不偿失的。



# 授人以渔



条件定义及处理：<https://dev.mysql.com/doc/refman/5.7/en/condition-handling.html>

存储过程函数：<https://dev.mysql.com/doc/refman/5.7/en/sql-syntax-data-definition.html>

游标：<https://dev.mysql.com/doc/refman/5.7/en/cursors.html>

控制流程：<https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html>



# 存储过程、函数区别

函数必须有返回值，存储过程没有，存储过程参数可以使用`IN、OUT、INOUT`类型，函数的参数必须是`IN`类型。



# 创建函数、存储过程

修改创建存储过程或函数的语法

```mysql
create procedure sp_name(proc_parameter[...])
	[characteristic...] routine_body
	
create function fun_name(func_paramter[...])
	return type
	[characteristic...] routine_body
```

解释：

**存储过程、函数参数**

```mysql
proc_parameter：[in|put|inout] param_name type

func_paramter:param_name type
```

**函数返回`type**`

​	任意MySQL数据类型

**`characteristic`特征**

```mysql
language sql	#说明下面过程的主题是使用sql准备的，有点鸡肋，mysql为今后sql外的其他语言准备
|[not] deterministic	#deterministic确定，表明函数的返回值完全由输入参数决定的
|{contains sql|not sql|reads sql data|modifies sql data}	
    #提供子程序使用的内在数据，目前只提供给服务器
    #contains sql表示子程序不包含读或写数据的语句
    #not sql表示子程序不包含sql语句
    #reads sql data表示子程序包含读数据的语句，不包含写语句
    #modifies sql data子程序包含写语句
    #默认使用contains sql
|sql security{definer|invoker}	#安全级别 definer定义者 invoker调用者
|comment 'string'		#注释				
```

**`routine_body`**

​	`SQL`代码的内容

MySQL允许存储过程、函数包含DDL语句（data define language）,允许存储过程中执行提交（commit）或者回滚（rooback），存储过程和函数不允许执行`load data infile`（快速导入数据）



栗子：

```mysql
#随机产生字符串
#定义新的命令结束符
delimiter $$
#DROP FUNCTION rand_string $$

CREATE FUNCTION rand_string(n INT)
RETURNS varchar(255)#返回字符串
BEGIN
	DECLARE char_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';	#定义变量
	DECLARE return_str VARCHAR(255) DEFAULT '';
	DECLARE i INT DEFAULT 0;
	WHILE i<n DO
		#CONCAT（）函数用于将多个字符串连接成一个字符串
		#floor：函数只返回整数部分，小数部分舍弃
		SET return_str=CONCAT(return_str,SUBSTRING(char_str, FLOOR(1+RAND()*52),1));
		SET i=i+1;
	END WHILE;
	RETURN return_str;
END $$
	
#随机int
delimiter $$
CREATE FUNCTION rand_int()
RETURN INT
BEGIN
	DECLARE return_str INT DEFAULT 10;
	SET return_str=RAND()*10+RAND()*100+RAND()*1000;
	RETURN return_str;
END $$


#插入数据
delimiter $$
#DROP FUNCTION insert_user $$
CREATE PROCEDURE insert_order(IN start INT(10),IN max_num INT(10))
BEGIN
DECLARE i int DEFAULT 0;
SET autocommit=0; #关闭自提交，需要手动提交
REPEAT
	SET i=i+1;
	INSERT INTO `order` VALUES((start+i),RAND()*10+RAND()*100+RAND()*1000,rand_string(10),10);
	UNTIL i=max_num
END REPEAT;
COMMIT;
END $$

delimiter ;
CALL insert_order(0,100000);
```



# 修改存储过程、函数

```mysql
alter{procedure|function} sp_name [characteristic...]
```

解释

**`characteristic`**

```mysql
{contains sql|not sql|reads sql data|modifies sql data}	
|sql security{definer|invoker}
|comment 'string'		#注释			
```



# 调用存储

```mysql
call sp_name[paramter[...]]
```



# 删除存储过程、函数

```mysql
drop {procedure|function} [if exists] sp_name
```

栗子：

```mysql
DROP PROCEDURE IF EXISTS user_order_account;
```



# 查看存储过程、函数

就是查看元数据

**查看存储过程或函数的状态**

```mysql
show {procedure|function} status [like 'pattern']
```

```mysql
show procedure status like 'file_in_stocl' \G
```

**查看存储过程或函数的定义**

```mysql
show create {procedure|function} sp_name
```

# 变量使用

**变量定义**

```mysql
declare dec_name type [default value]
```

**变量赋值**

直接赋值

```mysql
set var_name=...
```

粒子：

```mysql
SET return_str=CONCAT(return_str,SUBSTRING(char_str, FLOOR(1+RAND()*52),1));
```

**查询结果赋值**

注意：<span style="color:red">查询返回的结果必须只有一行</span>

```mysql
select colname[...] into var_name[...] table_expr
```

黎姿：

```mysql
#用存储过程遍历用户表，统计每一个用户自购订单数量，并存入统计表（新建一张统计表：用户id，自购单数）

#先查询每个用户的订单数叠加
#将查询的数据插入表中

#修改结束符
DELIMITER $$
#清空表数据
truncate table user_account;
#删除已存在的存储过程
DROP PROCEDURE IF EXISTS user_order_account;
CREATE PROCEDURE user_order_account(in minId INT,in maxId INT)
BEGIN
	DECLARE uid INT DEFAULT 0;
	DECLARE account INT DEFAULT 0;
	SET uid=minId;#设置循环用户ID最小id
	SET autocommit=0;#关闭自动提交事物
	WHILE uid<maxId DO
		SELECT SUM(o.order_account) into account FROM `order` AS o WHERE o.order_user_id=uid;
		INSERT INTO user_account VALUES(uid,account);
		SET uid=uid+1;	
	END WHILE;
	COMMIT;
END $$

CALL user_order_account(1,100000);

#检验 单独运行
SELECT COUNT(o.order_account) FROM `order` AS o WHERE o.order_user_id=3; 

```



**局部变量**

少用

```mysql
@xxx=yyy
```



# 定义条件和处理

一般用于游标

**条件的定义**

```mysql
declare condition_name condition for condition_value
```

解释：

```mysql
condition_value：
	sqlstate[value] sqlstate_value
	|mysql_error_code
```

**条件的处理**

```mysql
declare hander_type handler for condition_value[...] sp_statement
```

解释

```mysql
handler_type:
	continue
	exit
	undo
```

```mysql
condition_value:
	sqlstate[value] sqlsate_value
	|condition_name		#自定义的条件
	|sqlwarning			#01开头的sqlstate代码速记
	|not found			#02开头的sqlstate代码速记
	|sqlexception		
	|mysql_error_code
```



# 游标

声明游标：

```mysql
declare cursor_name cursor for select_statement
```

`opne`游标

```mysql
open cursor_name
```

`fetch`游标（取）

```mysql
fetch cursor_name into var_name[...]
```

`close`游标

```mysql
close cursor_name
```

**<span style="color:red">重点:</span>**

变量和条件必须在最前面，然后是右边，最后才是处理程序的声明



狸子：

```mysql
delimiter $$
#删除已存在的存储过程
DROP PROCEDURE if EXISTS add_account;
create PROCEDURE add_account(in add_account_val INT)
BEGIN
	#标志符
	DECLARE done boolean DEFAULT 0;
	DECLARE tmp int; #零时变量
	DECLARE add_cursor CURSOR for select id from user_account;	#声明一个游标
	DECLARE EXIT HANDLER for not found SET done=1;				#定义条件及条件处理
	
	#开启游标
	OPEN add_cursor;
	REPEAT
		FETCH add_cursor INTO tmp;
			UPDATE user_account u SET u.all_account=add_account_val WHERE u.id=tmp;
	UNTIL done END REPEAT;
	CLOSE add_cursor;
END $$

delimiter ;

CALL add_account(10);
```



# 控制流程

**if语句**

```mysql
IF search_condition THEN statement_list
    [ELSEIF search_condition THEN statement_list] ...
    [ELSE statement_list]
END IF
```

**case语句**

```mysql
CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list] ...
    [ELSE statement_list]
END CASE
```

或者

```mysql
CASE
    WHEN search_condition THEN statement_list
    [WHEN search_condition THEN statement_list] ...
    [ELSE statement_list]
END CASE
```



**loop语句**

```mysql
[begin_label:] LOOP
    statement_list
END LOOP [end_label]
```



**leave语句**

This statement is used to exit the flow control construct that has the given label. If the label is for the outermost stored program block, [`LEAVE`](https://dev.mysql.com/doc/refman/5.7/en/leave.html) exits the program.

[`LEAVE`](https://dev.mysql.com/doc/refman/5.7/en/leave.html) can be used within [`BEGIN ... END`](https://dev.mysql.com/doc/refman/5.7/en/begin-end.html) or loop constructs ([`LOOP`](https://dev.mysql.com/doc/refman/5.7/en/loop.html), [`REPEAT`](https://dev.mysql.com/doc/refman/5.7/en/repeat.html), [`WHILE`](https://dev.mysql.com/doc/refman/5.7/en/while.html)).

没想到这个渣渣威能看懂这个，哈哈哈

粒子：

```mysql
CREATE PROCEDURE doiterate(p1 INT)
BEGIN
  label1: LOOP
    SET p1 = p1 + 1;
    IF p1 < 10 THEN
      ITERATE label1;#循环遍历
    END IF;
    LEAVE label1;#退出循环，使用return也可以
  END LOOP label1;
  SET @x = p1;
END;
```



**iterate语句**

表示结束当前循环，进入下一个循环

[`ITERATE`](https://dev.mysql.com/doc/refman/5.7/en/iterate.html) can appear only within [`LOOP`](https://dev.mysql.com/doc/refman/5.7/en/loop.html), [`REPEAT`](https://dev.mysql.com/doc/refman/5.7/en/repeat.html), and [`WHILE`](https://dev.mysql.com/doc/refman/5.7/en/while.html) statements. [`ITERATE`](https://dev.mysql.com/doc/refman/5.7/en/iterate.html) means “start the loop again.”



**while语句**

```mysql
[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label]
```







