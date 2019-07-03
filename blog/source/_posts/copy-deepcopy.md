---
title: Python 深拷贝/浅拷贝和赋值
date: 2017-05-24 21:38:11
tags:
  - Copy
  - Deepcopy
categories:
  - python
---

#### 概念说明
- 赋值：对象的赋值是进行对象引用（内存地址）传递，引用的地址没有变化，相当于设置了另外一个别名；注意：当修改不可变类型的值时，对应内部的引用发生变化；其中可变类型(列表，字典)，不可变类型(数字，字符串，元祖)
- 浅拷贝：只拷贝父对象，不会拷贝对象的内部的子对象,只是原对象元素的引用，下列操作相当于浅拷贝
  	- 使用切片操作[:]
  	- 使用list/dir/set
  	- 使用copy.copy()
- 深拷贝：拷贝对象及其子对象,创建一个新的对象，不使用原来的对应对象引用


#### 实例

##### 赋值

```python
In [1]: l1 = [1, 2, 3, [4, 5], (7,8)]
 
In [2]: l2 = l1
 
In [3]: id(l1), id(l2)
Out[3]: (4360956240, 4360956240)	# 指向相同的内存地址
 
In [4]: l1[3].append(6)
 
In [5]: l1
Out[5]: [1, 2, 3, [4, 5, 6], (7, 8)]
 
In [6]: l2
Out[6]: [1, 2, 3, [4, 5, 6], (7, 8)]
 
In [7]: id(l1), id(l2)
Out[7]: (4360956240, 4360956240)
 
In [8]: l2[0] = 0
 
In [9]: l2
Out[9]: [0, 2, 3, [4, 5, 6], (7, 8)]
 
In [10]: l1
Out[10]: [0, 2, 3, [4, 5, 6], (7, 8)]
 
In [11]: l1 is l2
Out[11]: True
 
In [12]: id(l1[3])
Out[12]: 4360955808
 
In [13]: id(l2[3])
Out[13]: 4360955808
```
##### 浅拷贝

```python
import copy
 
l1 = [3, [66, 55, 44], (7, 8, 9)]
 
# l2 = list(l1)	# 相当于浅拷贝操作,l1 l2代表不同的列表，但两者引用同一个列表和元祖，如下图一
l2 = copy.copy(l1)
 
print '=================《流畅的python》第217页 浅拷贝================='
print 'id-----',id(l1[1]), id(l2[1])	# 4318639744 4318639744
print 'id-----',id(l1), id(l2)	# 4318640104 4318555600
 
l1.append(100) # [3, [66, 55, 44], (7, 8, 9), 100]
l1[1].remove(55) #  [3, [66, 44], (7, 8, 9), 100]
print 'l1---->', l1 # [3, [66, 44], (7, 8, 9), 100]
print 'l2---->', l2 # [3, [66, 44], (7, 8, 9)]
print '============================================='
l2[1] += [33, 22] # [3, [66, 44, 33, 22], (7, 8, 9)]
l2[2] += (10, 11) # [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
print 'l1====>', l1 # [3, [66, 44, 33, 22], (7, 8, 9), 100]
print 'l2====>', l2 # [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
print '================= 如下图二===================='
```
![图一](https://ttxsgoto.github.io/img/copy/copy01.png)
![图二](https://ttxsgoto.github.io/img/copy/copy02.png)

##### 深拷贝

```python
import copy
 
class Bus(object):
 
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)
 
    def pick(self, name):
        self.passengers.append(name)
 
    def drop(self, name):
        self.passengers.remove(name)
 
bus1 = Bus(['AAA', 'BBB', 'CCC'])
bus2 = copy.copy(bus1)
bus3 = copy.deepcopy(bus1)
 
print id(bus1), id(bus2), id(bus3)
# 4318715792 4318757008 4318757072
 
bus1.drop('AAA')
print bus2.passengers
# [u'BBB', u'CCC']
print id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
# 4560506320 4560506320 4560590464 bus2是bus1的浅复制的副本,所以id相同
print bus3.passengers
# [u'AAA', u'BBB', u'CCC']
```






