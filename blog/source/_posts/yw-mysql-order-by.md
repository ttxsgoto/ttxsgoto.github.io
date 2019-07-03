---
title: Order by工作原理
date: 2018-12-28 22:46:51
tags:
  - 排序
categories:
  - Mysql
---
#### 全字段排序
mysql会给每个线程分配一块内存用于排序，称为sort_buffer

sort_buffer_size 是mysql为排序开辟的内存大小(sort_buffer)大小，如果要排序的数据量小于sort_buffer_size，排序在内存中完成，如果排序数据量太大，大于该值，则需要用磁盘临时文件辅助排序

#### rowid排序
全字段排序只对原表数据读取一次，剩下的都在sort_buffer和临时文件中进行，如果查询返回的字段很多，那么sort_buffer里面存放的字段数太多，内存里放下的数据行数就很少，需要分成很多个临时文件，排序的性能会受影响，这时需要使用rowid排序

max_length_for_sort_data 字段用来控制排序的行数据的长度的参数，如果单行的长度超过这个值，mysql会换一种算法，即将查询的列和主键id放入sort_buffer中

这种算法需要多访问一次表的主键索引

如果内存大，mysql就会多利用内存，尽量减少磁盘访问

**写入有序数据，通过索引本身就是写入数据就是有序的，那么order by 就不需要排序了，直接就是对应的数据**











