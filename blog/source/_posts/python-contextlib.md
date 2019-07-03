---
title: Python Contextlib模块
date: 2017-04-11 22:32:21
tags:
  - Contextlib
categories:
  - python
---
#### 说明
contextlib是为了加强with语句，提供上下文机制的模块，它是通过Generator实现的。通过定义类以及写__enter__和__exit__来进行上下文管理;
contextlib中有nested和closing，前者用于创建嵌套的上下文，后则用于帮你执行定义好的close函数.
```python
#!/usr/bin/env python
#coding:utf-8
  
class WithinContext(object):
    def __init__(self,context):
        print "WithinContext.__init__(%s) " %context
 
    def do_something(self):
        print "WithinContext.do_something()"
 
    def __del__(self):
        print "WithinContext.__del__"
 
 
class Context(object):
    def __init__(self):
        print "Context.__init__()"
 
    def __enter__(self):
        """
        在主体代码执行前执行
        """
        print "Context.__enter__()"
        return WithinContext(self)
 
    def __exit__(self,exc_type,exc_val,exc_tb):
        """
        在主体代码执行后执行
        """
        print "Context.__exit__()"
 
 
with Context() as c :
    """
    as后面的变量是在__enter__函数中返回的
    """
    c.do_something()
#执行结果：
Context.__init__()
Context.__enter__()
WithinContext.__init__(<__main__.Context object at 0x7f95045167d0>) 
WithinContext.do_something()
Context.__exit__()
WithinContext.__del__
```
contextlib中的contextmanager作为装饰器来提供一种针对函数级别的上下文管理机制.
```python
#!/usr/bin/env python
#coding:utf-8
 
import contextlib
 
@contextlib.contextmanager
def make_context():
    print "entering"
    try:
        yield {}
    finally:
        print "exiting"
         
with make_context() as value:
    print "inside with statement:", value
     
#执行结果：
"""
entering
inside with statement: {}
exiting
"""
 
@contextlib.contextmanager
def make_context(name):
    print "entering",name
    yield name
    print "exiting",name
     
with contextlib.nested(make_context('A')) as (c):
    print "inside with statement: ",c
     
#执行结果：
"""
entering A
inside with statement:  ['A']
exiting A
"""

with contextlib.nested(make_context('A'),make_context("B"),make_context("C")) as (A,B,C):
    """
    nested用于创建嵌套的上下文
    """
    print "inside with statement: ",A,B,C
     
#执行结果：
"""
entering A
entering B
entering C
inside with statement:  A B C
exiting C
exiting B
exiting A
"""
 
class Door(object):
    def __init__(self):
        print "__init__()"
    def close(self):
        print "close()"
        return
          
with contextlib.closing(Door()) as door:
    """
    closing执行定义好的close函数
    """
    print "inside with statement."
     
#执行结果：
"""
__init__()
inside with statement.
close()
"""
```



