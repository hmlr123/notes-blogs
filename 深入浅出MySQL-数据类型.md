---
title: 深入浅出MySQL-数据类型
date: 2019-04-01 11:03:58
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



`MySQL`的数据类型，主要包括数值型、字符串类型、日期和时间。不同版本`MySQL`类型不同

本章节较为简单，重点关注常用数据类型以及注意事项内容

<!--more-->

# 英语词汇

binary			二进制

hexadecimal	    十六进制

decimal		     小数、十进制

tiny			    微小的，很少的

medium		   中等

precision		  精度

unsigned		 无符号

# 数值类型

严格数值数据类型`(INTEGER、SMALLINT、DECIMAL和NUMERIC)`，以及近似数值数据类型`(FLOAT、REAL和DOUBLE PRECISION)`

![](https://i.loli.net/2019/04/01/5ca18add31895.jpg)

![](https://i.loli.net/2019/04/01/5ca18adb9bd88.png)

注意事项：

* 当我们指定长度时，比如int(5)表示当数值宽度小于5位是，前面将被填充，默认int int(11)。一般配合zerofill使用，表示用0填充
* 可选属性`unsigned`表示无符号。适用于非负数，或需要较大上限值时。他的取值范围，下限0，上限为原来的两倍，例如`tinyint`有符号-128~+127，无符号0~255。如果一个列指定为`zerofill`,MySQL默认自动为该列添加该属性
* `Auto_increment`自增长，一个表只允许一个字段又该属性
* `decimal`在MySQL内部是字符串存储，比浮点数更精确，适用于货币等精度高的 数据
* M表示精度，有多少位数（整数+小数）D表示标度，小数点后面位数
* 在没有制定进度的情况，浮点数会存储实际精度，但是这个精度有实际的硬件和系统控制，不同情况可能有所不同。`decimal`不指定精度，默认整数位10，小数位0。溢出会`waring`，多出的部分被截掉。
* `Bit`默认1





# 日期类型

![](https://i.loli.net/2019/04/01/5ca190c977c10.png)



注意事项：

* 如果经常插入或者更新数据为当前系统时间，用时间撮，不同时区，用时间撮，显示的时间不同。时区时间转换

* `timestamp`不插入值是，系统毁人创建`current_timestamp` ，MySQL只给表中的第一个`timestamp`字段设置为默认时间，第二个默认为0

* 查看当前时间 就是上一节说的吃查询元数据

  ```mysql
  show variables like 'time_zone'
  ```

* 注意时间表示范围，`timestamp`取值范围为1970.01.01. 08：00：01到2038年某一天，不适合存放很久的日期
* 日期的分割符没有强制限制，2019-04-01、2019-4-1、20190401、201941、2019+04+01...都可以





# 字符串类型

![](https://i.loli.net/2019/04/01/5ca1922d607a5.png)

注意事项：

* 在检索的时候，char列删除尾部的空格，varchar保留空格



## 枚举类型`ENUM`

字符串类型

1. 创建表，`gender`字段为枚举类型

   ```mysql
   create table t (gender enum('M','F'));
   ```

2. 插入记录

   ```mysql
   insert into t values('M'),('n'),('1'),(null);
   ```

注意事项：

* `enum`是忽略大小写的
* `enum`只允许从值集合中选取单个值。不能一次多个值



## SET类型

![](https://i.loli.net/2019/04/01/5ca194ff7df25.jpg)