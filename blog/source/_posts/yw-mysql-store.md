---
title: 运维 Mysql分析性能（存储过程）
date: 2017-04-08 22:24:31
tags:
  - Mysql
categories:
  - Mysql
---
#### 描述
之前有一次故障问题，查看数据库慢查询日志，从日志和存储过程本身直接分析，不能得知问题点，之后通过profiling来分析，记录如下
#### 分析
0. 查看慢查询日志，发现，出现大量执行等待，使用profiling来分析语句问题点
1. 先打开profiling： SET profiling=1;
2. 手动执行有问题的语句： call  GetRoleList(75760);   
3. SHOW profiles;  可以查看到每条对应SQL语句执行时间，对于执行慢的sql这部分是需要优化的;
4. 执行完毕后，关闭 SET profiling=0;

