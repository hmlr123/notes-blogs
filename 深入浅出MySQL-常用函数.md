---
title: 深入浅出MySQL-常用函数
date: 2019-04-06 09:56:03
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---



很多函数很有用，多百度，不要死记硬背

<!--more-->

# 授人以渔

<https://dev.mysql.com/doc/refman/5.5/en/date-and-time-functions.html>

相关函数请看官方文档，比我这还详细，谷歌翻译是个好东西。



这里只说个人觉得应该注意的事项。

# 字符串函数

![](https://i.loli.net/2019/04/06/5ca80efece3c2.jpg)

![](https://i.loli.net/2019/04/06/5ca80f000884b.png)

* `concat()`，任何字符串和null连接的结果都将是null
* `left(str,x)`和`right(str,x)`入轨第二个函数为`null`，将不返回字符串

# 数值函数

![](https://i.loli.net/2019/04/06/5ca81056d07c7.jpg)

补充：

`sum()`函数：返回指定字段的数据之和

`count()`函数：返回指定字段的数据的行数（记录数）

`avg()`函数：平均数

* `round(x,y)`函数：如果是整数，将保留y为数量的0，如果没有y，默认0，即将x四舍五入后取整。
* `truncate`仅仅只是截断，不进行四舍五入

# 日期和时间函数

![](https://i.loli.net/2019/04/06/5ca80efecc203.jpg)

![](https://i.loli.net/2019/04/06/5ca80efebed05.jpg)

# 流程函数

![](https://i.loli.net/2019/04/06/5ca80eff7e0c3.png)

# 其他常用函数

![](https://i.loli.net/2019/04/06/5ca80efeca07d.jpg)

