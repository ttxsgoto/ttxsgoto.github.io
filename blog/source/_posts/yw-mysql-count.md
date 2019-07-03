---
title: Count计数变慢
date: 2018-12-28 22:48:50
tags:
  - count
categories:
  - Mysql
---

#### count(*) 实现方式
不同的引擎，实现方式不同
- Myisam把一个表的总行存在磁盘上，执行时直接返回这个数
- Innodb，需要把数据一行行的从引擎中读出来，然后累积计数，遍历全表


#### 不同的count用法
count()为聚合函数，对于返回的结果集，一行行判断，如果count函数的参数不为null，累计值加1，最终返回累计值
- count(*)
- count(id)
- count(字段)
- count(1)

这里count(*),count(id),count(1)返回满足条件的结果集的总行数，而count(字段)返回满足条件不为null的总数
性能比较原则：
1. server层要做什么就给什么
2. innodb只给必要的值

#### 效率排序

count(字段)< count(id)<count(1)=count(*)