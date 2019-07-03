---
title: Python Decorator装饰器
date: 2017-05-13 10:57:35
tags:
  - Decorator
categories:
  - python
---
### 功能
给程序去重，在不改动源代码和原有调用方式下，给函数添加额外的功能模块（如验证功能）

### 特性
	- 能把装饰的函数替换成其他函数
	- 装饰器在加载/导入模块时立即执行,被装饰的函数在被调用的时候运行

### 变量作用域
- 内部函数,不修改全局变量可以访问全局变量
- 内部函数,修改同名全局变量,则python会认为它是一个局部变量
- 在内部函数修改同名全局变量之前调用变量名称, 则引发UnboundLocalError

```python
#!/usr/bin/env python
# coding: utf-8
 
b = 6
 
def f1(a):
    print(a)
    # b = 4
    print(b)
    b = 4
    print(b)
 
f1(3)
 
# UnboundLocalError: local variable 'b' referenced before assignment
"""
对于上面的说明:
“Python 编译函数的定义体时，它判断 b 是局部变量，因为在函数中给它赋值了。生成的字节码证实了这种判断，Python 会尝试从本地环境获取 b。后面调用 f1(3) 时， f1 的定义体会获取并打印局部变量 a 的值，但是尝试获取局部变量 b 的值时，发现 b 没有绑定值。”
摘录来自: [巴西] Luciano Ramalho. “流畅的Python”。
"""
 
b = 6
 
def f2(a):
    global b    # 函数中b为全局变量
    print(a)
    print(b)
    b = 4
    print(b)
 
f2(3)
```

### 闭包
闭包指延伸了作用域的函数，其中包含函数定义体中引用，但是不在定义体中定义的非全局变量，函数是不是匿名的没有关系，关键是它能访问定义体之外定义的非全局变量

最常用功能： 闭包使得局部变量在函数外被访问成为可能

自由变量(freevariable): 指未在本地作用域中绑定的变量，也就是上面所说的局部变量

通过函数名.code.co_freevars来获取自由变量对应的名称 通过函数名.closure[0].cell_contents 来获取自由变量的值
```python
def add(x):
    def inner(y):
        return x + y
    return inner
 
add1 = add(5)
print(add1(10))     # 15
print(add1.__code__.co_freevars)    # ('x',)
print(add1.__closure__[0].cell_contents)    # 5
```
	- 当函数离开创建环境后,依然持有其上下文状态
	- 要形成闭包，首先需要一个嵌套的函数，即函数中定义了另一个函数，子函数引用父函
	  数的变量,子函数称为闭包;
	- Python中函数也是对象，所以函数也有很多属性，和闭包相关的属性是 
	  __closure__,__closure__ 属性定义的是一个包含 cell 对象的元组，其中元组
      中的每一个cell对象用来保存作用域中变量的值

``` python
#!/usr/bin/env python
#coding: utf_8
 
def func(n):
    print 'id(n)----> {}'.format(id(n))
 
    def new_power(x):
        return x**n
    print 'id(new_power)----> {}'.format(id(new_power))
    return new_power
 
first = func(5) # 在调用func函数时产生一个闭包new_power,并且已拥有局部变量n的值,即使func生命周期结束后,值n依然存在,因为n被new_power引用,所以不会被回收
 
print id(first)
del func
print first(2)
print first.__closure__, first.__closure__[0].cell_contents # 函数也为对象,其中的闭包属性
##### 运行结果 #####
id(n)----> 140676064892712
id(new_power)----> 4357115728
4357115728
32
```

### 理解
装饰器就是执行一个函数,当执行到@auth时，内部的动作为：
	- 执行auth函数，并将@auth下面的函数作为auth函数的参数，即@auth == auth(f1)
	- 将执行完的auth函数返回值赋给@auth下面的函数的函数名，即 f1 = auth(f1),相当于执行inner函数
### 装饰器实例

#### 函数(无参数)的装饰器
```python
def auth(func):
    def inner():
        print "before"    #执行函数前执行动作
        func()
        print "after"    #执行函数后执行动作
    return inner
     
@auth  #@auth ==> f1 = auth(f1) ==>f1() 相当于执行inner函数，func为f1函数
def f1():
    print "This is f1 function"
 
f1()
 
#### 运行结果 ####
before
This is f1 function
after
```
#### 函数含有(一个/多个)参数的装饰器
```python
def auth_arg(func):
    def inner(*args, **kwargs):  # 传递一个/多个参数
        print "before"
        ret1,ret2 = func(*args, **kwargs)  # 相当于f2(*args, **kwargs)函数
        print "after"
        return ret1,ret2
    return inner
 
@auth_arg
def f2(*args, **kwargs):
    print  args,kwargs
    return args,kwargs
 
key1 = 'ttxsgoto'
dict1 = {}
dict1['ttxs'] = 'goto'
ret1, ret2 = f2(key1, **dict1)
print ret1, ret2

#### 运行结果 ####
before
('ttxsgoto',) {'ttxs': 'goto'}
after
('ttxsgoto',) {'ttxs': 'goto'}
```

#### 有参数的装饰器
```python
def auth_arg(list1=[]):
    if not list1:
        print 'None ....'
        raise Exception('LIST is  Null ')
    new_list = []
    for li in list1:
        li += 'a'
        new_list.append(li)
    print new_list
    def inner(func):
        print 'Before'
        def in_inner(*args, **kwargs):
            ret1, ret2 = func(*args, **kwargs)
            return ret1, ret2
        return in_inner
    return inner
 
@auth_arg(list1=['a', '1'])
def f3(*args, **kwargs):
    print "f3 ---", args, kwargs
    return args, kwargs
 
a = [1,2]
dict1={}
dict1['ttxs'] = 'goto'
 
print f3(a, **dict1)
 
#### 运行结果 ####
['aa', '1a']
Before
f3 --- ([1, 2],) {'ttxs': 'goto'}
```
#### 多装饰器
- 在foo函数上层包裹了一层w1，又包裹了一次w2，一个嵌套一个函数，执行
- 可用于登录后再判断有没有权限，可以使用两个装饰器

```python
def w1(func):
    def inner():
        print "before01"
        func()
        print "after01"
    return inner
 
def w2(func):
    def inner():
        print "before02"
        func()
        print "after02"
    return inner
 
@w2
@w1
def foo():
    print "foo"
 
foo() #先执行w1，在执行w2，嵌套执行，foo = w2(w1(foo)))
#### 运行结果 ####
before02
before01
foo
after01
after02
```
#### 函数的类装饰器(1)
```python
class Foo(object):
    '''
    类装饰器主要依靠类的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法
    '''
 
    def __init__(self, func):
    	print "__init__  function"
        self.func = func
 
    def __call__(self, *args, **kwargs):
        print "Before..."
        self.func()
        print "After..."
 
@Foo
def bar():
    print 'bar funtion()'
 
bar()
 
#### 运行结果 ####
__init__  function
Before...
bar funtion()
After...
```
#### 函数的类装饰器(2)
```python
class Decorate(object):
 
    def __init__(self,arg_list=[]):
        self.arg_list = arg_list
 
    def __call__(self, func):
        if not self.arg_list:
                print 'None ....'
                raise Exception('LIST is  Null ')
        new_list = []
        for li in self.arg_list:
            li += 'a'
            new_list.append(li)
        print new_list
        def inner(*args, **kwargs):
            ret1, ret2 = func(*args, **kwargs)
            return ret1, ret2
        return inner
 
@Decorate(arg_list=['a', 'b'])
def f3(*args, **kwargs):
    print "f3  function", args, kwargs
    return args, kwargs
 
a = [1,2]
dict1={}
dict1['ttxs'] = 'goto'
ret = f3(a, **dict1)
print ret
 
#### 运行结果 ####
['aa', 'ba']
f3  function ([1, 2],) {'ttxs': 'goto'}
(([1, 2],), {'ttxs': 'goto'})
```
### 装饰器的不足
使用装饰器极大地复用了代码，缺点就是原函数的元信息丢失
```python
from functools import wraps
 
def auth(func):
    @wraps(func)	#保持原函数信息一致，如果没有该装饰器，返回为inner函数
    def inner():
        print "before"
        func()
        print "after"
    return inner
 
@auth
def f1():
    print "This is f1 function"
 
f1()
print f1, f1.__name__
#### 运行结果 ####
before
This is f1 function
after
<function inner at 0x10e55b140> inner	#这些显示为inner函数,本来调用的是f1函数

```






