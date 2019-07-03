---
title: Redo log 和 Binlog
date: 2018-12-25 19:43:38
tags:
  - binlog
categories:
  - Mysql
---
WAL(Write-Ahead-Logging),先写日志，再写磁盘

- redo log(重做日志)
    - 存储引擎层面
    - 存储容量大小固定，循环写
    - checkpoint 当前要擦除的位置，擦除记录前需要将数据更新到数据文件中
    - write-pos 当前写入记录的位置
    - 功能：保证InnoDB即使数据库发生异常重启，之前提交的记录也不会丢失(crash-safe)
    - 插件式，innodb引擎特有
    - 物流日志-记录在某个数据页上做了什么修改
- binlog(归档日志)
    - Server层面
    - 逻辑日志，记录的是这个语句的原始逻辑，sql语句是什么样子的


执行器和引擎执行update操作流程：

1. 先取出该查询，如果在内存中则直接返回给执行器，否则先从磁盘读入内存，然后返回
2. 执行器拿到引擎给的行数据，执行相关操作，再调用引擎接口写入这行新数据
3. 引擎将这行新数据更新到内存中，同时将这个更新记录到redo log中，此时redo log处于prepare状态，然后告知执行器执行完成，随时可以提交事务
4. 执行器生成这个操作的binlog，将binlog写入磁盘
5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改写提交(commit)状态，更新完成

流程如下：
```
获取数据 ——> 返回数据 ——> 执行逻辑 ——> 写入新行 ——> 新行更新到内存 ——> 写入redo log处于prepare状态 ——> 写binlog ——> 提交事务redo log处于commit状态完成
```

redo log和binlog都可以用于表示事务的提交状态，而两阶段提交就是让这两个状态保持逻辑上的一致

数据一致性保持理解：

1 prepare阶段 
2 写binlog 
3 commit

当在2之前崩溃时
重启恢复：后发现没有commit，回滚。备份恢复：没有binlog 一致
当在3之前崩溃
重启恢复：虽没有commit，但满足prepare和binlog完整，所以重启后会自动commit。备份：有binlog. 一致

**参数配置**
innodb_flush_log_at_trx_commit 参数设置成1时，表示每次事务的redo log都直接持久化到磁盘，保证mysql异常重启后数据不丢失
sync_binlog参数设置成1时，表示每次事务的binlog都持久化到磁盘，保证mysql异常重启后binlog不丢失

