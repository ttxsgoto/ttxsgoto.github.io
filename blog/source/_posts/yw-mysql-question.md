---
title: Mysql 出现You can't specify target table for update in FROM clause错误的解决方法
date: 2018-12-03 20:22:02
tags:
  - Mysql
categories:
  - Mysql
---
#### 问题描述：
在同一个sql语句中，先select同一个表的某些值，然后再update这个表

如执行如下sql语句：
```python
update logistic_statistic set fluid_id='cca03b6a372045f2891fef62d9524652' where id in (
  select id from logistic_statistic where company= '新能源有限公司');
```
执行报错：
```python
Error : You can't specify target table 'logistic_statistic' for update in FROM clause
```

#### 解决方法：
select的结果再通过一个中间表select多一次，就可以避免这个错误

执行如下sql语句：
```python
update logistic_statistic set fluid_id="cca03b6a372045f2891fef62d9524652" where id in ( 
  select other_id from ( 
    SELECT id as other_id from logistic_statistic where company= '新能源有限公司') as a ;
  );
```





