---
title: Python Descriptor描述符01
date: 2018-01-31 20:24:28
tags:
  - Descriptor
categories:
  - python
---
### 描述符说明
	- 只要是定义了__get__()、__set()__、__delete()__中任意一个方法的对象都叫描述符
	- 描述符是一个具有绑定行为的对象属性，其属性的访问被描述符协议方法覆写
	- 通常来说Python对象的属性控制默认是这样的：从对象的字典(__dict__)中
	  获取（get），设置（set）,删除（delete），比如：对于实例a，a.x的查找
      顺序为a.__dict__['x'],然后是type(a).__dict__['x'].如果还是没找
      到就往上级(父类)中查找最后查找是否有__getattr__
	- 如果属性x是一个描述符，那么访问a.x时不再从字典__dict__中读取，而是调用
	  描述符的__get__()方法，对于设置和删除也是同样的原理
	- 同时含有__get__,__set__方法,为数据描述符对象;仅实现__get__方法,为非数据描述符对象;
	- 数据描述符优先于实例的字典，对于相同名字的会覆盖;实例的字典优先于非数据描述符,但不会覆盖;

### 调用方法
    - 描述符作为属性访问时,是被自动调用
    - 类属性描述符对象,使用type.__getattribute__,把Class.x转换成Class.__dict__['x'].__get__(None, Class)
    - 实例属性描述符对象,使用object.__getattribute__,把object.x转换为type(object).__dict__['x'].__get__(object, type(object))

### 描述符用途,使用场景
    - 当希望对某些类的属性进行特别的处理而不会对整体的其他属性有影响的话,可以使用修饰符
    - 只要一个类实现了__get__,__set__,__delete__方法的任意一种就是描述符
    - 描述符会'劫持'那些本是self.__dict__的操作
    - 把一个类的操作托付给另外一个类
    - 静态方法,类方法,parperty都是构建描述符的类

### 实例

#### 实例一

```python
# 属性进行相应的处理而不会对整体有影响
 
class Integer(object):
 
    def __init__(self, age):
        self.age = age
 
    def __get__(self, instance, owner):
        print '__get__-----', self, instance, owner
        if instance is None:
            return self
        # return instance.__dict__[self.age]  # 获取dict中对应的属性值
        return self.age
 
    def __set__(self, instance, value):
        print '__set__-----', self, instance, value
        if value < 0 or type(eval(str(value))) == float:
            raise ValueError('Age must int and not negative ')
        # instance.__dict__[self.age] = value # 修改dict中对应的属性值
        self.age = value
 
    def __del__(self):
        del self.age
        pass
 
 
class SexType(object):
 
    def __init__(self, sex):
        self.sex = sex
 
    def __get__(self, instance, owner):
        if instance is None:
            return self
        # return instance.__dict__[self.sex]  # 获取dict中对应的属性值
        return self.sex
 
    def __set__(self, instance, value):
        if value not in ['M', 'W']:
            raise ValueError('The value must be M/W ')
        # instance.__dict__[self.sex] = value # 修改dict中对应的属性值
        self.sex = value
 
    def __del__(self):
        del self.sex
        pass
 
 
class Person(object):
    age = Integer('age')
    sex = SexType('sex')
 
    def __init__(self, name, sex, age):
        self.name = name
        self.sex = sex
        self.age = age
 
    @property
    def info(self):
        return 'Person info --name:{},--sex:{},--age:{}'.format(self.name, self.sex, self.age)
 
A = Person(name='ttxsgoto', sex='W', age= 15 )
 
print A.__dict__
print Person.__dict__
print A.info
 
########## 结果 ##########
'''
{'name': 'ttxsgoto'}
{'info': <property object at 0x1028ad418>, '__module__': '__main__', 'age': <__main__.Integer object at 0x1028b30d0>, 'sex': <__main__.SexType object at 0x1028b3110>, '__dict__': <attribute '__dict__' of 'Person' objects>, '__weakref__': <attribute '__weakref__' of 'Person' objects>, '__doc__': None, '__init__': <function __init__ at 0x1028b2578>}
__get__----- <__main__.Integer object at 0x1028b30d0> <__main__.Person object at 0x1028b3150> <class '__main__.Person'>
Person info --name:ttxsgoto,--sex:W,--age:15
'''
```

#### 实例二

```python
# 数据描述符和非数据描述符的区别
 
class Access(object):
 
    def __init__(self, var=None, name='var'):
        self.var = var
        self.name = name
 
    def __get__(self, instance, owner):
        print '__get__----', self, instance, owner, self.name
        return self.var
 
    def __set__(self, instance, value):  # 含有__set__方法为数据描述符
        print '__set__----', self, instance, value
        self.var = value
 
 
class MyClass(object):
    x = Access(10, 'ttxsgoto')
 
    def __init__(self, x):
        self.x = x
 
t = MyClass(100)
print '----\n', t.x
print '====\n', t.__dict__
print 'xxxx\n', MyClass.__dict__
print 'yyyy\n', MyClass.x
 
########### 数据描述符的输出 ###########
'''
__set__---- <__main__.Access object at 0x10687c2d0> <__main__.MyClass object at 0x10687c610> 100
----
__get__---- <__main__.Access object at 0x10687c2d0> <__main__.MyClass object at 0x10687c610> <class '__main__.MyClass'> ttxsgoto
100
====
{}
xxxx
{'__module__': '__main__', '__dict__': <attribute '__dict__' of 'MyClass' objects>, 'x': <__main__.Access object at 0x10687c2d0>, '__weakref__': <attribute '__weakref__' of 'MyClass' objects>, '__doc__': None, '__init__': <function __init__ at 0x10687b320>}
yyyy
__get__---- <__main__.Access object at 0x10687c2d0> None <class '__main__.MyClass'> ttxsgoto
100
'''
 
########### 非数据描述符的输出 ###########
'''
----
100
====
{'x': 100}
xxxx
{'__module__': '__main__', '__dict__': <attribute '__dict__' of 'MyClass' objects>, 'x': <__main__.Access object at 0x1012502d0>, '__weakref__': <attribute '__weakref__' of 'MyClass' objects>, '__doc__': None, '__init__': <function __init__ at 0x10124f050>}
yyyy
__get__---- <__main__.Access object at 0x1012502d0> None <class '__main__.MyClass'> ttxsgoto
10
'''
```