---
title: 深入浅出MySQL-运算符
date: 2019-04-06 09:04:57
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



so easy

<!--more-->

# 算数运算符

![](https://i.loli.net/2019/04/06/5ca7fc0a168d5.png)

注意事项：

* 当除法、取余、DIV、MOD操作的时候，参数2为0或者为null，结果为null



# 比较运算符

![](https://i.loli.net/2019/04/06/5ca7fcb67ddfd.png)

注意事项：

* 比较运算符用于数字、字符串、表达式。**数字作为浮点数比较，字符串不区分大小写比较**
* `between`用法 `a between b and c`；当a、b、c类型相同的时候直接比较，不同的时候先转换数据类型，在比较。
* `REGEXP`用法：`str REGEXP str_pat`



# 逻辑运算符

![](https://i.loli.net/2019/04/06/5ca7fe17b5117.png)



# 位运算符

![](https://i.loli.net/2019/04/06/5ca7fe4d0b080.png)



# 运算符优先级

![](https://i.loli.net/2019/04/06/5ca7ff15acfc0.jpg)



* 注意`:=`含义：表示赋值
* 区别`=`和`:=`区别，在大多数情况下`=`表示相等的意思，就是比较运算符。只有在`set`或者`update`情况下，才是赋值的意思
* `<=>`表示左边等于右边，右边等于左边，和`=`区别，`<=>`可以用于null的比较，而`=的`null比较是无效的