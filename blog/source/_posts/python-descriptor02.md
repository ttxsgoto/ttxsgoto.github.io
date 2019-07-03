---
title: Python Descriptor描述符02
date: 2018-02-01 22:30:30
tags:
  - Descriptor
categories:
  - python
---
#### 说明
- 使用描述符时，实例对象的属性访问会触发描述符的\_\_get\_\_方法
- 使用描述符时，实例对象的属性赋值会触发描述符的\_\_set\_\_方法
- 通过obj.\_\_dict\_\_[xxx]=yyy 赋值会跳过描述符
- 没有\_\_get\_\_方法的覆盖描述符，给对象的属性赋值会触发\_\_set\_\_方法，读取属性时会直接从实例中返回新赋予的值，而不会返回描述符对象， 也就是说读取操作实例属性会遮盖描述符，直接从\_\_dict\_\_中获取
- 非数据描述符中，如果设置了同名的实例属性，描述符会被遮盖，致使描述符无法处理那个实例的那个属性,即获取属性的值将直接通过\_\_dict\_\_中获取
- 类属性赋值能覆盖描述符属性，如果想控制设置类属性的操作，需要把描述符依附在类的类上，即依附在元类上
- 类中定义的函数属于绑定方法，因为用户定义的函数都有get方法，所以依附到类上，相当于描述符，为非覆盖性描述符

#### 概念描述
- 描述符类： 实现描述符协议的类
- 托管类： 把描述符实例声明为类属性的类
- 描述符实例： 描述符类的各个实例，声明为托管类的类属性
- 托管实例：托管类的实例
- 储存属性： 托管实例中存储自身托管属性的属性
- 托管属性： 托管类中由描述符实例处理的公开属性，值存储在储存属性中， 也就是说，描述符实例和储存属性为托管属性建立了基础

#### 描述符实例

```python
class Quantity:
    """ 描述符类 """
    __counter = 0
 
    def __init__(self):
        cls = self.__class__
        prefix = cls.__name__
        index = cls.__counter
        self.storage_name = '_{}#{}'.format(prefix, index)
        cls.__counter += 1
 
    def __get__(self, instance, owner):
        """
        获取对应的属性时, 调用该方法
        :param instance: 描述符实例
        :param owner: 托管类的引用(type),通过描述符从托管类中获取属性时用得到
        :return:
        """
        if instance is None:
            return self
        return getattr(instance, self.storage_name)
 
    def __set__(self, instance, value):
        """
        :param instance: 描述符实例
        :param value: 托管实例设置的值
        :return:
        """
        if value > 0:
            setattr(instance, self.storage_name, value)
        else:
            raise ValueError('value must be > 0.')
 
 
class LineItem:
    """ 托管类 """
    weight = Quantity() # 描述符实例
    price = Quantity()   # 描述符实例
    # weight, price 为储存属性
 
    def __init__(self,desc, weight, price):
        self.desc = desc
        self.weight = weight
        self.price = price
        # self.weight, self.price 为托管属性
 
    def subtotal(self):
        return self.weight * self.price
 
# 描述符实例为引用类的类属性
 
line = LineItem('test01', 12, 12.3) # 托管实例
 
print(line.weight)  # 实例对象的属性访问会触发描述符的__get__方法
 
print(line.subtotal())
 
print(LineItem.weight)
```

#### 描述符实例重构
```python
import abc
 
 
class AutoStorage:
    __counter = 0
 
    def __init__(self):
        cls = self.__class__
        prefix = cls.__name__
        index = cls.__counter
        self.storage_name = '_{}#{}'.format(prefix, index)
        cls.__counter += 1
 
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage_name)
 
    def __set__(self, instance, value):
        setattr(instance, self.storage_name, value)
 
 
class Validated(abc.ABC, AutoStorage):
    """ 验证相关模块 """
 
    def __set__(self, instance, value):
        """ 把验证操作委托给validate方法"""
        value = self.validate(instance, value)
        super(Validated, self).__set__(instance, value)
 
    @abc.abstractmethod
    def validate(self, instance, value):
        """ 抽象方法 """
        pass
 
 
class Quantity(Validated):
    def validate(self, instance, value):
        if value <= 0:
            raise ValueError('value must be > 0')
        return value
 
 
class NonBlank(Validated):
    def validate(self, instance, value):
        value = value.strip()
        if len(value) == 0:
            raise ValueError('value cannot be empty or blank')
        return value
 
 
class LineItem:
    """ 托管类 """
    weight = Quantity()  # 描述符实例
    price = Quantity()  # 描述符实例
    desc = NonBlank()  # 描述符实例
 
    # weight, price 为储存属性
 
    def __init__(self, desc, weight, price):
        self.desc = desc
        self.weight = weight
        self.price = price
        # self.weight, self.price 为托管属性
 
    def subtotal(self):
        return self.weight * self.price
 
 
line = LineItem('abc', 12, 12.3)
print(line.subtotal())
```
#### 覆盖型和非覆盖型描述符对比
- 覆盖型描述符 实现了\_\_set\_\_方法的描述符,实现了set方法会覆盖对实例属性的赋值操作

- 非覆盖型描述符 没有实现\_\_set\_\_方法的描述符是非覆盖型描述符，如果设置了同名的实例属性，描述符会被遮盖，致使描述符无法处理那个实例的那个属性

-  类属性赋值能覆盖描述符属性，如果想控制设置类属性的操作，需要把描述符依附在类的类上，即依附在元类上

```python
def cls_name(obj_or_cls):
    cls = type(obj_or_cls)
    if cls is type:
        cls = obj_or_cls
    return cls.__name__.split('.')[-1]
 
 
def display(obj):
    cls = type(obj)
    if cls is type:
        return '<class {}>'.format(obj.__name__)
    elif cls in [type(None), int]:
        return repr(obj)
    else:
        return '<{} object>'.format(cls_name(obj))
 
 
def print_args(name, *args):
    pseudo_args = ','.join(display(x) for x in args)
    print('->{}.__{}__({})'.format(cls_name(args[0]), name, pseudo_args))
 
 
class Overriding:
    """ 数据描述符 """
 
    def __get__(self, instance, owner):
        print_args('get', self, instance, owner)
 
    def __set__(self, instance, value):
        print_args('set', self, instance, value)
 
 
class OverridingNoGet:
    """ 没有 __get__方法的覆盖型描述符 """
 
    def __set__(self, instance, value):
        print_args('set', self, instance, value)
 
 
class NonOverriding:
    """ 非数据描述符"""
 
    def __get__(self, instance, owner):
        print_args('get', self, instance, owner)
 
 
class Managed:
    over = Overriding()
    over_no_get = OverridingNoGet()
    non_over = NonOverriding()
 
    def span(self):
        print('-> Managed.spam({})'.format(display(self)))
 
 
################覆盖型描述符##################
"""获取和设置值 都是通过描述符的get和set方法完成"""
obj = Managed()
obj.over
# Overriding.__get__(<Overriding object>,<Managed object>,<class Managed>)
Managed.over
# Overriding.__get__(<Overriding object>,None,<class Managed>)
 
obj.over = 7
# Overriding.__set__(<Overriding object>,<Managed object>,7)
obj.over
# Overriding.__get__(<Overriding object>,<Managed object>,<class Managed>)
 
obj.__dict__['over'] = 8  # 跳过描述符,通过obj.__dict__赋值
print(vars(obj))
# {'over': 8}
print(obj.over)
# None
obj.over  # obj.over描述符仍会覆盖取值(obj.over)这个操作
# Overriding.__get__(<Overriding object>,<Managed object>,<class Managed>)
 
 
##########没有__get__()方法的覆盖型描述符##########
"""没有__get__方法的覆盖描述符,给对象的属性赋值会触发__set__方法,读取属性时会直接从实例中返回新赋予的值,而不会返回描述符对象,读取操作实例属性会遮盖描述符"""
print(obj.over_no_get)
# <__main__.O
# verridingNoGet object at 0x102973358>
print(Managed.over_no_get)
# <__main__.OverridingNoGet object at 0x102973358>
obj.over_no_get = 7
# OverridingNoGet.__set__(<OverridingNoGet object>,<Managed object>,7)
print(obj.over_no_get)
# <__main__.OverridingNoGet object at 0x101873358>
obj.__dict__['over_no_get'] = 9
print(obj.over_no_get)	# 通过类属性取值,没有通过描述符,因为没有__get__方法
# 9
obj.over_no_get = 7
# OverridingNoGet.__set__(<OverridingNoGet object>,<Managed object>,7)
print(obj.over_no_get)  # 只要有同名的实例属性,描述符会被遮盖,默认的实例属性获取方法遮盖描述符
# 9
 
 
########## 非覆盖型描述符 ##########
"""
非数据描述符中，如果设置了同名的实例属性，描述符会被遮盖，致使描述符无法处理那个实例的那个属性
"""
 
obj1 = Managed()
obj1.non_over
# NonOverriding.__get__(<NonOverriding object>,<Managed object>,<class Managed>)
obj1.non_over = 7
print(obj1.non_over)	# 实例属性赋值会遮盖描述符__get__方法
# 7
Managed.non_over
# NonOverriding.__get__(<NonOverriding object>,None,<class Managed>)
del obj1.non_over
obj1.non_over	# 删除实例属性后,又从描述符__get__方法中获取
# NonOverriding.__get__(<NonOverriding object>,<Managed object>,<class Managed>)
 
################################
# 在类中覆盖描述符
################################
obj2 = Managed()
 
# 覆盖类中的描述符属性
Managed.over = 1
Managed.over_no_get = 2
Managed.non_over = 3
 
print(obj2.over, obj2.over_no_get, obj2.non_over)
# 1 2 3
```
#### 描述符用法
1. 使用特性以保持简单
2. 只读描述符必须有\_\_set\_\_方法，如果需要实现只读属性，__get__和__set__方法必须都实现，否则实例的同名属性会遮盖描述符，只读属性的\_\_set\_\_方法只需抛出AttributeError异常，并提供合适的错误信息
3. 用于验证的描述符可以只有\_\_set\_\_方法
4. 仅有__get__方法的描述符可以实现高效缓存，同名实例属性会遮盖描述符
5. 非特殊的方法可以被实例属性遮盖， 函数和方法都只实现了__get__方法，不会处理同名实例属性的赋值操作

以上内容学习记录参照《流畅的python》 第20章 属性描述符
