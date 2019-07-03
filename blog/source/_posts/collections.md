---
title: Python Collections模块
date: 2017-07-02 20:53:00
tags:
  - Collections
categories:
  - python
---
Python拥有内置的数据类型，比如str, int, list, tuple, dict；collections模块在这基础上进行了扩展，使用更加灵活，常用方法整理如下：

#### namedtuple
功能:将名称映射到序列的元素上 namedtuple是不可变的;用于将元祖的位置访问转换为通过名称访问,使代码易读；
常用于将csv/sqlite3中得到的大型元祖列表,通过元素来访问数据,容易出错,这时将返回的元祖转换为命名元祖就很有必要
```python
#!/usr/bin/env python
# encoding: utf-8
from collections import namedtuple
import csv
 
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')  # typename: 元组名称 field_names: 元祖字段名称
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print emp.name, emp.title
 
import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print emp.name, emp.title
 
dict_website=[('a','www.a.com','aaa'),('b','www.b.com','bbb'),('c','www.c.com','ccc')]
name_websit = namedtuple('sites',['name','www','finder'])
for website in dict_website:
    print  name_websit._make(website)
 
Subscriber = namedtuple('Sub', 'addr, joined, name')
list_demo = ('cd', '2017-07-01', 'ttxsgoto')
print Subscriber._make(list_demo)
ttxsgoto = Subscriber('ttxs', '2017-07-01', 'goto')
print ttxsgoto
print ttxsgoto.addr, ttxsgoto.joined, len(ttxsgoto)
########## 结果 ##########
"""
sites(name='a', www='www.a.com', finder='aaa')
sites(name='b', www='www.b.com', finder='bbb')
sites(name='c', www='www.c.com', finder='ccc')
Sub(addr='cd', joined='2017-07-01', name='ttxsgoto')
Sub(addr='ttxs', joined='2017-07-01', name='goto')
ttxs 2017-07-01 3
"""
```
#### Counter
功能:统计序列中元素出现的次数
most_common(n): 统计出现次数,从高到低,依次排列个数
```python
#!/usr/bin/env python
# encoding: utf-8
from collections import Counter
 
default_list = ['a', 'b', 'b', 1, 1, 3]
c = Counter(default_list)
 
print c, c.most_common(2)
########## 结果 ##########
# Counter({1: 2, 'b': 2, 'a': 1, 3: 1}) [(1, 2), ('b', 2)]
```
#### OrderedDict
功能:指定字典中的顺序,根据添加顺序排序,大小为普通dict的2倍多
因为添加了额外的链表,如果涉及大量数据,需要考虑本身占用的内存
```python
#!/usr/bin/env python
# encoding: utf-8
from collections import OrderedDict
 
items = (
    ('a',1),
    ('b',2),
    ('c',3),
)
 
default_dict = dict(items)
order_dict = OrderedDict(items)
 
print default_dict, order_dict
########## 结果 ##########
# {'a': 1, 'c': 3, 'b': 2} OrderedDict([('a', 1), ('b', 2), ('c', 3)])

```
#### defaultdict
功能:带有默认值的字典, 一键多值的字典
```python
#!/usr/bin/env python
# encoding: utf-8
from collections import defaultdict
 
members = (
    ['male', 'John'],
    ['male', 'Jack'],
    ['female', 'Lily'],
    ['male', 'Pony'],
    ['female', 'Lucy'],
)
 
result_list = defaultdict(list)  # 默认列表
result_dict = defaultdict(dict)  # 默认字典
result_set = defaultdict(set)    # 默认集合
 
for sex, name in members:
    result_list[sex].append(name)  # 列表
    result_dict[sex] = name        # 字典
    result_set[sex].add(name)      # 集合
 
print result_list, result_dict, result_set
########## 结果 ##########
"""
defaultdict(<type 'list'>, {'male': ['John', 'Jack', 'Pony'], 'female': ['Lily', 'Lucy']})
defaultdict(<type 'dict'>, {'male': 'Pony', 'female': 'Lucy'})
defaultdict(<type 'set'>, {'male': set(['John', 'Pony', 'Jack']), 'female': set(['Lily', 'Lucy'])})
"""
```

#### 参考链接
https://docs.python.org/2/library/collections.html#module-collections












