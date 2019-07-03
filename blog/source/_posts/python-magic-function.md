---
title: Python 特殊方法
date: 2018-02-11 21:13:59
tags:
  - class
categories:
  - python
---
### 说明
主要说明类的特殊方法(也叫魔法方法)，了解使用场景和方法，使用时方便查询

### 具体说明
#### 创建实例和销毁
##### \_\_new\_\_
##### \_\_init\_\_
##### \_\_del\_\_
```python
class Foo01:
    def __new__(cls, *args, **kwargs):
        """对象实例化调用的第一个方法,在__init__之前, args,kwargs传递给init方法"""
        print('__new__')
        return super().__new__(cls)
 
    def __init__(self, name):
    	"""初始化实例操作"""
        print('__init__')
        self.name = name
 
    def __del__(self):
    	""" del 时调用，或实例自行销毁时调用"""
        print('__del__')
        return
 
 
foo = Foo01(name='ttxs')  # __init__
print(foo.name)  # ttxs
del foo  # __del__
```
#### 属性管理
##### \_\_getattr\_\_
##### \_\_getattribute\_\_
##### \_\_setattr\_\_
##### \_\_delattr\_\_
##### \_\_dir\_\_
```python
class Foo02:
    def __getattr__(self, item):
        """ 该方法在访问一个不存在的属性时调用 """
        print('__getattr__')
        if item not in self.__dict__:
            return None
        return super(Foo02, self).__getattr__(item)
 
    def __setattr__(self, key, value):
        """ 该方法在对属性进行赋值和修改时调用,应该避免"无限递归"错误,如: self.name = 'xxx' """
        print('__setattr__')
        # self.key = value    # 每一次属性赋值时, __setattr__都会被调用，因此不断调用自身导致无限递归
        self.__dict__[key] = value
        return super(Foo02, self).__setattr__(key, value)
 
    def __delattr__(self, item):
        """ 该方法在删除属性时调用 """
        print('__delattr__')
        # del self.item         # 无限递归, 原因同上
        return super(Foo02, self).__delattr__(item)
 
    def __getattribute__(self, item):
        """ 该方法在属性被访问时调用,调用__getattr__前必定会调用 __getattribute__ """
        print('__getattribute__')
        return super(Foo02, self).__getattribute__(item)
 
 
foo02 = Foo02()
foo02.abc = 'abc'  # 调用__setattr__方法
print(foo02.abc)  # 调用__getattribute__方法
print('error---', foo02.x)  # 调用__getattr__方法
foo02.__dict__  # 调用__getattribute__方法
del foo02.abc  # 调用__delattr__方法
```
#### 序列表示形式
##### \_\_str\_\_
##### \_\_repr\_\_
##### \_\_bytes\_\_
##### \_\_doc\_\_
##### \_\_format\_\_
```python
class Foo03:
    """ 文档展示说明表示 (__doc__)"""
 
    def __init__(self, name, id):
        self.name = name
        self.id = id
 
    def __repr__(self):
        """ 主要针对程序的调试,调用和使用,如果定义了str函数print会调用str,没有就调用repr函数 """
        print('__repr__')
        return '%s with %s id' % (self.name, self.id)
 
    def __str__(self):
        """print方法时调用"""
        print('__str__')
        return self.name.capitalize()
 
    def __bytes__(self):
        print('__bytes__')
        return '%s in bytes is %s' % (self.name, self.id)
 
    def __format__(self):
        """ 调用format方法时 """
        print('__format__')
        return '%s in format is %s' % (self.name, self.id)
 
 
foo03 = Foo03(name='ttxsgoto', id=666)
print(Foo05.__doc__)    # 说明文档信息
print(foo05)            # __str__()方法
 
f = Foo03(name='ttxsgoto', id=666)
f.__format__()
ff = bytes(f)  # bytes调用
```
#### 集合管理
##### \_\_len\_\_
##### \_\_getitem\_\_
##### \_\_setitem\_\_
##### \_\_delitem\_\_
##### \_\_contains\_\_
```python
class Foo04:
    def __init__(self, value):
        if value is None:
            self.value = {}
        else:
            self.value = value
 
    def __len__(self):
        """表示集合长度"""
        return len(self.value)
 
    def __getitem__(self, item):
        """执行self[item],调用该方法"""
        print('__getitem__')
        return self.value[item]
 
    def __setitem__(self, key, value):
        """执行self[key]=value,调用该方法"""
        print('__setitem__')
        self.value[key] = value
 
    def __delitem__(self, key):
        """执行 del self[key],调用该方法"""
        print('__delitem__')
        del self.value[key]
 
    def __contains__(self, item):
        """判断item是否在容器中,调用该方法in/not in,如 item in container"""
        print('__contains__')
        if item in self.value:
            return True
        return False
 
 
foo04 = Foo04(value={'a': 'a'})
foo04['xx'] = 'abc'  # 调用__setitem__方法
print(foo04['xx'])  # 调用__getitem__方法
print(len(foo04))  # 调用__len__方法
 
if 'xx' in foo04:  # 调用__contains__方法
    print('ok')
else:
    print('error')
```
#### 迭代枚举
##### \_\_iter\_\_
##### \_\_next\_\_
```python
import re
import reprlib
from collections import abc
 
RE_WORD = re.compile('\w+')
 
class Foo04:
    """ 通过索引从文本中提取单词 """
 
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)
 
    def __iter__(self):
        """ 可迭代对象 """
        return SentenceIterator(self.words)
 
    def __len__(self, index):
        return len(self.words)
 
    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)  # 用于生成大型数据结构的简略字符串表示形式, 默认情况下最多30个字符
 
 
class SentenceIterator:
    """ 迭代器 """
    def __init__(self, words):
        self.words = words
        self.index = 0
 
    def __next__(self):
        """ 返回下一个可用的元素，如果没有，抛出StopIteration异常 """
        try:
            word = self.words[self.index]
        except IndexError:
            raise StopIteration()
        self.index += 1
        return word
 
    def __iter__(self):
        """ 返回self，以便在应该使用可迭代对象的地方使用迭代器 """
        return self
 
 
foo04 = Foo04(text='"The time has come," the Walrus said,')
 
print(foo04)
print(iter(foo04))
for foo in foo04:
    print(foo)
 
print(issubclass(SentenceIterator, abc.Iterator))
```
#### 上下文管理
##### \_\_enter\_\_
##### \_\_exit\_\_
参考链接
https://ttxsgoto.github.io/2017/04/11/python-contextlib/
#### 描述符
##### \_\_get\_\_
##### \_\_set\_\_
##### \_\_delete\_\_
参考链接
https://ttxsgoto.github.io/2018/01/31/descriptor/
https://ttxsgoto.github.io/2018/02/01/python-descriptor02/
