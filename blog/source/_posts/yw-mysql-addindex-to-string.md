---
title: 字符串加索引
date: 2018-12-28 20:41:50
tags:
  - Index
categories:
  - Mysql
---

#### 前缀索引
前缀字符长度索引，所占空间更小，同时会增加额外的记录扫描次数

在建立索引时需要关注的是区分度，区分度越高越好

查看对应索引的区分度方法
```python
select count(distinct a) as L from user;
select count(*) as total from user;
 
select count(distinct left(a, 4) as L4,
       count(distinct left(a, 5) as L5,
from user
```
**注意**：使用前缀索引就用不上覆盖索引查询对查询性能的优化，这是在选择是否使用前缀索引时需要考虑的因素

#### 倒序存储
对前面n位数都是重复的字段，可以使用倒序存储，将存储的内容倒序存储
```python
select field from t where word=reverse('input_string');
```

#### 使用hash字段
在表中再创建一个整数字段，来保存字段的校验码，同时在这个字段创建索引
```python
alter table t add word_crc int unsigned, and index(word_crc);
```

#### 总结
- 直接创建完整索引，这样可能会占用大量的空间
- 创建前缀索引，节省空间，但会增加查询扫描次数，并且不能使用覆盖索引
- 倒序存储，在创建前缀索引，用于绕过字符串本身前缀区分度不高的问题
- 创建hash字段索引，查询性能稳定，有额外的存储和计算消耗，同时不支持范围扫描

