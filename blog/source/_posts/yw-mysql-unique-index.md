---
title: 普通索引和唯一索引的区别
date: 2018-12-20 20:11:46
tags:
  - Index
categories:
  - Mysql
---
#### 查询操作
- 查找到满足条件的第一个记录，在查找下一个记录，直到到不满足记录时停止
- 唯一索引，找到第一个满足条件记录，停止检索

#### 更新操作
change buffer:如果数据页没有在内存中，innodb会将更新操作缓存在change buffer中，这样不需要从磁盘中读取数据页；下次查询访问这个数据页时，将数据页读入内存，然后执行change buffer中和这个页有关的操作

- 唯一索引更新操作不能使用change buffer，因为判断记录是否存在，必须将数据页读入内存，只有普通索引更新操作可以使用change buffer
- change buffer使用的是buffer pool里的内存，大小可以通过参数**innodb_change_buffer_max_size**来设置

#### 插入操作
1. 记录要更新的目标页在内存中，此时插入操作基本一致，从内存中读取数据对应的位置，执行插入语句
2. 记录要更新的目标页不在内存中
    - 唯一索引，需要将数据页读入到内存，判断有没有冲突，执行语句
    - 普通索引，将更新记录到change buffer，执行语句

将数据从磁盘读入内存涉及到随机IO访问，成本高；change buffer减少了随机磁盘访问，所以对更新性能的提升更明显


#### change buffer使用场景
适用于写多读少的业务场景，写入后不立即读取，常见业务系统如日志、账单类系统

如果业务更新后马上需要做查询，即更新先记录change buffer，之后查询这个数据页，会立刻触发merge操作，这样随机访问io次数不会减少，反而增加了change buffer的维护代价


#### 索引选择
- 查询性能方面无差异
- 更新操作，因为有change buffer机制，所以普通索引更新操作性能更好


#### change buffer && redo log
- redo log 主要节省的是随机写磁盘IO消耗(转成顺序写)
- change buffer主要节省的是随机读磁盘io的消耗

