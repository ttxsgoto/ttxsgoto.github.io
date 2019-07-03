---
title: Python Datetime模块
date: 2017-04-10 21:51:17
tags:
  - Datetime
categories:
  - python
---
日常工作中常用到时间处理，一般使用datetime模块来解决，datetime模块包括一些函数和类，用于完成日期和时间的解析、格式化和相关的运行.

#### 时间转换为字符串(格式化输出)
```python
datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
# strftime参数，strftime(format[, tuple]) -> string
python中时间日期格式化符号：
%y 两位数的年份表示（00-99）
%Y 四位数的年份表示（000-9999）
%m 月份（01-12）
%d 月内中的一天（0-31）
%H 24小时制小时数（0-23）
%I 12小时制小时数（01-12）
%M 分钟数（00-59）
%S 秒（00-59）
%a 本地简化星期名称
%A 本地完整星期名称
%b 本地简化的月份名称
%B 本地完整的月份名称
%c 本地相应的日期表示和时间表示
%j 年内的一天（001-366）
%p 本地A.M.或P.M.的等价符
%U 一年中的星期数（00-53）星期天为星期的开始
%w 星期（0-6），星期天为星期的开始
%W 一年中的星期数（00-53）星期一为星期的开始
%x 本地相应的日期表示
%X 本地相应的时间表示
%Z 当前时区的名称
%% %号本身
```
#### 字符串和datetime转换
```python
'''
把str转换为datetime,转换方法是通过datetime.strptime()实现，需要一个日期和时间的格式化字符串
'''
In [1]: from datetime import datetime
 
In [2]: print datetime.now()
2017-04-10 21:10:03.619051
 
In [3]: type(datetime.now())
Out[3]: datetime.datetime
 
In [4]: str = '2017-04-10 21:13:14'
 
In [5]: type(str)
Out[5]: str
 
In [6]: daytime = datetime.strptime(str,'%Y-%m-%d %H:%M:%S')
 
In [7]: daytime
Out[7]: datetime.datetime(2017, 4, 10, 21, 13, 14)
 
In [8]: print daytime
2017-04-10 21:13:14
 
In [9]: type(daytime)
Out[9]: datetime.datetime
```

#### data和datetime转换
```python
# datetime ——> date
In [1]: import datetime
In [2]: datetime.datetime.now().date()
Out[2]: datetime.date(2017, 6, 20)
 
# date ——> datetime
In [5]: today = datetime.date.today()

In [6]: datetime.datetime.combine(today, datetime.time())
Out[6]: datetime.datetime(2017, 6, 20, 0, 0)
In [7]: datetime.datetime.combine(today, datetime.time.min)
Out[7]: datetime.datetime(2017, 6, 20, 0, 0)
```
#### 当前时间
```python
'''
datetime是模块，datetime模块还包含一个datetime类，通过from datetime import datetime导入。
如果仅导入import datetime，则需要使用datetime.datetime.now()返回当前日期和时间，其类型是datetime
'''
In [1]: from datetime import datetime
 
In [2]: now = datetime.now()
 
In [3]: now
Out[3]: datetime.datetime(2017, 4, 10, 20, 11, 7, 241762)
 
In [4]: print now
2017-04-10 20:11:07.241762
 
In [5]: type(now)
Out[5]: datetime.datetime
```
#### 特定时间
```python
In [1]: from datetime import datetime
 
In [2]: day = datetime(2017,4,10,13,14)
 
In [3]: day
Out[3]: datetime.datetime(2017, 4, 10, 13, 14)
 
In [4]: print day
2017-04-10 13:14:00
```
#### 时间加减
```python
'''
对日期和时间进行加减实际上就是把datetime往后或往前计算，得到新的datetime。加减可以直接用+和-运算符，需要导入timedelta
'''
In [1]: from datetime import datetime,timedelta
 
In [2]: now = datetime.now()
 
In [3]: now
Out[3]: datetime.datetime(2017, 4, 10, 21, 39, 8, 413031)
 
In [4]: now+timedelta(hours=2)
Out[4]: datetime.datetime(2017, 4, 10, 23, 39, 8, 413031)
 
In [5]: now-timedelta(days=2)
Out[5]: datetime.datetime(2017, 4, 8, 21, 39, 8, 413031)
 
In [6]: now+timedelta(days=1,hours=1)
Out[6]: datetime.datetime(2017, 4, 11, 22, 39, 8, 413031)
```


