---
title: Python 动态属性和特性
date: 2018-01-30 22:42:28
tags:
  - property
categories:
  - python
---

###  \_\_getattr\_\_ 方法

\_\_getattr\_\_(self, name) 作用仅当没有指定名称name的属性时才调用\_\_getattr\_\_方法
```python
from collections import abc
import keyword
 
class FrozenJSON:
    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += '_'
            self.__data[key] = value
 
    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        else:
            return FrozenJSON.build(self.__data[name])
 
    @classmethod
    def build(cls, obj):  # 备选构造器
        if isinstance(obj, abc.Mapping):  # 映射
            return cls(obj)
        elif isinstance(obj, abc.MutableSequence):  # 列表
            return [cls.build(item) for item in obj]
        else:
            return obj
 
if __name__ == '__main__':
    grad = FrozenJSON({'name': 'ttxs', 'class': 123})
    print(grod.name)
    print(grad.class_)
```

### @property 解析
- property 本身是一个类
- property(fget=None, fset=None, fdel=None, doc=None)
所有的参数都是可选，如果没有把函数传给某个参数，那么得到的特性对象就不允许执行相应的操作

```python
class LineItem:
 
    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price
 
    def subtotal(self):
        return self.weight * self.price
 
    def get_weight(self):
        return self.__weight
 
    def set_weight(self, value):
        if value > 0:
            self.__weight = value
        else:
            raise ValueError('value must be > 0')
 
    weight = property(get_weight, set_weight, )
 
xxx = LineItem('ttxs', 10, 12.5)
print(xxx.subtotal())
xxx.weight=-20
print(xxx.subtotal())
```

### 处理属性的重要属性和函数
- \_\_class\_\_ : 对象所属类的引用(obj.\_\_class\_\_  和type(obj)相同)

- \_\_getattr\_\_：只在对象的类中寻找，而不在实例中寻找
- \_\_dict\_\_： 存储对象或类的可写属性,有dict属性的对象，任何时候都能设置新属性
- \_\_slots\_\_： 类可以定义这个属性，限制实例能有哪些属性
- dir([object])：列出对象的大部分属性
- getattr(obj, name[, defult])：从obj对象中获取name字符串对应的属性
- hasattr(obj, name)：如果obj对象中存在指定的属性，或者能以某种方式通过obj对象获取指定的属性，返回true
- setattr(obj, name, value)：把obj对象指定属性的值设置为value，前提是obj对象能接受这个值；这个函数可能会创建一个新的属性或者覆盖现有的属性
- vars([obj])：返回obj对象的\_\_dict\_\_属性，如果实例所属的类定义了\_\_slots\_\_属性，实例没有\_\_dict\_\_属性，那么vars函数不能处理那个实例

### 处理属性的特殊方法
- obj.attr 和 getattr(obj, 'attr', 42) 都会触发 Class.\_\_getattribute\_\_(obj, 'attr')方法
- \_\_delattr\_\_(self, name) del 语句， 都会触发Class.\_\_delattr\_\_(obj, 'attr')方法
- \_\_dir\_\_ 把对象传给dir函数时调用，列出属性
- \_\_getattr\_\_(obj, name) 仅当获取指定的属性失败，搜索过obj，Class和超类之后调用
- \_\_getattribute\_\_(self, name)尝试获取指定的属性时总会获取调用这个方法，寻找的属性是特殊属性或者特殊方法时除外
- \_\_setattr\_\_(self, name, value) 尝试设置指定的属性时总会调用这个方法，点号和setattr内置函数会触发这个方法
