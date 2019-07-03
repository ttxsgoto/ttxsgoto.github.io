---
title: Django mgirate 错误
date: 2018-09-17 19:37:18
tags:
  - migrate
categories:
  - Django
---
#### 问题描述
最近在工作中，django model添加表时，如果有外键，在migrate时常常出现如下报错：
```python
File "/root/.virtualenvs/lib/python3.5/site-packages/pymysql/connections.py", line 1057, in _read_query_result
    result.read()
  File "/root/.virtualenvs/lib/python3.5/site-packages/pymysql/connections.py", line 1340, in read
    first_packet = self.connection._read_packet()
  File "/root/.virtualenvs/lib/python3.5/site-packages/pymysql/connections.py", line 1014, in _read_packet
    packet.check_error()
  File "/root/.virtualenvs/lib/python3.5/site-packages/pymysql/connections.py", line 393, in check_error
    err.raise_mysql_exception(self._data)
  File "/root/.virtualenvs/lib/python3.5/site-packages/pymysql/err.py", line 107, in raise_mysql_exception
    raise errorclass(errno, errval)
django.db.utils.IntegrityError: (1215, 'Cannot add foreign key constraint')
```
#### 原因分析
查看资料后，网上大体出现这样报错的原因有：
- 外键对应的字段数据类型不一致
- 两张表的存储引擎不一致
- 设置外键时“删除时”设置为“SET NULL”

发现这3点都不是引起我们报错的原因，首先我们的外键的字段都是UUID类型一样， 存储引擎也同样都是InnoDB,外键使用默认的django.db.models.deletion.CASCADE，这3点都不是造成错误的原因；

后来通过反复比较， 发现发现新建的表和建立外键的关系表中对应的字符集(default character set)和默认集合(default collation)这两者的类型不一致，导致外键关系创建不上,修改成一致后，再migrate，问题解决
所以 在这里应该再加一条，
- 在导入新库时，对应的字符集和集合类型应该和原数据库一致

#### 总结
在开发环境中，常把线上的数据库导入开发环境中， 这样本地数据库的字符集和集合类型有可能和线上的就不一致，导致上述问题； 
上述问题的原因： 线上数据库在创建数据库时指定的字符集为(utf8mb4)集合类型为(utf8mb4_general_ci),而在本地创建数据库时指定的字符集类型为(utf8mb4)集合类型为(utf8mb4_bin); 尤其在多人协作时这样的问题更容易出现，所以这样的操作时，应先查看数据库对应的存储引擎、字符集和集合类型等信息，避免出现意想不到的错误。






