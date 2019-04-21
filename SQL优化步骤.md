---
title: SQL优化步骤
date: 2019-04-21 12:38:31
tags:
  - MySQL
categories:
  - 读书笔记
  - 深入浅出MySQL
comment: true
---

一次实验报告，感觉还行，挂上来

<!--more-->



# 查看慢查询参数

查看慢查询

![](https://i.loli.net/2019/04/21/5cbbf49ba2a40.png)

开启慢查询

![](https://i.loli.net/2019/04/21/5cbbf4bb846a6.png)

**重启慢MySQL客户端，才能生效**



# 设置慢查询时间

查看慢查询时间

![](https://i.loli.net/2019/04/21/5cbbf4efe2035.png)



设置慢查询时间

![](https://i.loli.net/2019/04/21/5cbbf5269e0ee.png)



# 查看表结构，在没有索引的列查询



查看表结构：

![](https://i.loli.net/2019/04/21/5cbbf58ed117b.png)

可以看到user_id是主键索引，user_weixin是普通索引。

查看没有索引的字段数据

![](https://i.loli.net/2019/04/21/5cbbf5ad09e96.png)



# 统计慢查询次数

统计慢查询次数：

![](https://i.loli.net/2019/04/21/5cbbf5cf07732.png)

可以看出超过0.01秒的查询有三个（即慢查询）



# 分析慢查询语句

分析语句：

Explain语句分析，查看mysql执行情况

![](https://i.loli.net/2019/04/21/5cbbf5fb34995.png)

可以看出这个查询没有用索引，全表扫描，扫描了100288行。

使用profile分析：查看profile是否开启

![](https://i.loli.net/2019/04/21/5cbbf64372f8e.png)

发现profile没有开启，开启profile

![](https://i.loli.net/2019/04/21/5cbbf6655e97d.png)

使用profile分析

![](https://i.loli.net/2019/04/21/5cbbf6a8a61cf.png)

发现操作三耗时最久，

查看操作3各部分消耗时间：



![](https://i.loli.net/2019/04/21/5cbbf6cb25e37.png)

可以发现sending data消耗最高。

使用trace分析：分析mysql优化器对语句的执行情况，由于我的版本低于5.6，不带有 这个功能，所以跳过。



# 建立索引

对user_zfb建立索引，在分析查询时间：

![](https://i.loli.net/2019/04/21/5cbbf715f0449.png)

再次查询：

![](https://i.loli.net/2019/04/21/5cbbf735511f9.png)

可以发现这个是根据索引查询，扫描一行，效率提高了

再用profile分析查询时间

![](https://i.loli.net/2019/04/21/5cbbf7506a0cf.png)

发现加了索引后，操作6，7消耗时间明显降低了很多。

在查看慢查询有哪些：

增加了一次，发现是修改表结构建立索引之前的，即相同语句慢查询没有增加

![](https://i.loli.net/2019/04/21/5cbbf76c74f01.png)

# 总结

明确三个工具的作用：

Profile查看你所有的操作。

Explain查看查询语句的执行情况，特别是多表操作的时候。

Trace查看mysql优化器怎么优化你的sql语句。

 

注意事项：

每次关闭mysql客户端，profiling都会关闭，需要自己手动开启。

设置慢查询时间后需要重启客户端才行。