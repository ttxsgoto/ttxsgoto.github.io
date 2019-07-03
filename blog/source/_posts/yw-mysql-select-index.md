---
title: 索引选择
date: 2018-12-21 11:14:08
tags:
  - Index
categories:
  - Mysql
---
#### 优化器选择索引逻辑
- 扫描行数
- 使用临时表
- 是否排序


#### 基数
一个索引上不同值的个数，称之为“基数(cardinality)”;基数越大，索引的区分度越好

show index from x 查看索引对应的基数

- 采样统计， 通过设置参数innodb_stats_persistent来设置
    ```
    设置为on，表示统计信息会持久化存储，默认值N为20，M为10
    设置为off, 表示统计信息只存储在内存中，默认值N为8，M为16
    ```
analyze table t # 用于重新统计索引信息，当发现explain中rows值和实际情况差距比较大时，可以采用使用该命令来处理

#### 索引选择异常和处理

- 使用force index强制选择一个索引
	```
    select * from t force index(a) where a between 1000 and 2000;
    ```
- 修改查询语句
- 新建更合适的索引，来提供优化器来选择，或者删掉误用的索引



















