---
title: Python 生成器、迭代器、反射器
date: 2017-02-09 22:37:32
tags:
  - Yeild
  - Generator
categories:
  - python
---
### 生成器（generator)

一个函数调用时返回一个迭代器，那么这个就叫生成器（generator），如果函数中包含yield语法，那么这个函数就变成了生成器
return作用：在一个生成器中，如果没有return，则默认执行到函数完毕；如果遇到return，如果在执行过程中return则会抛出StopIteration终止迭代

```python
def fab(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
for n in fab(5):
    print n
```
**yield：** 每需要一个时，添加一个
简单地讲，yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，调用 fab(5) 不会执行 fab 函数，而是返回一个 iterable 对象！在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。


### 迭代器 

**迭代器iter** :存在内存中或者文件中，一次只能读取1个元素
是访问元素的一种方式，迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完后结束

迭代器的优点：不需要事项准备好整个迭代过程中的所有元素，迭代器仅在迭代到某个元素时才计算该元素，在这之前或之后，元素可以不存在或者被销毁，该特点使它特别适合用于遍历一个大的文件或集合，如几个G的文件

**特点**

    1.访问值不需要关心迭代器内部结构，仅需通过next()方法来不断取下一个内容
    2.不能随机访问集合中的某个值，只能从头到尾依次访问
    3.访问到一半时不能回退
    4.用于循环大的数据集合，节约内存

**方法**

    next():返回迭代器的下一个元素
    __iter__:返回迭代器对象本身

 **定义一个迭代器**
 
    names = iter(["a","b","d"])
**使用**
    
    print (names.next())


### 反射器

- hasattr(obj,attr)    这个方法用来检查obj中是否有一个名为attr的属性，返回一个布尔值
- getattr(obj,attr)     这个方法将返回名为attr值得属性的值，如attr为’bar’，则返回obj.bar方法
- setattr(obj,attr,val)    调用这个方法将给obj的名为attr的值得属性赋值为val，如果attr为’bar’，则相当于obj.bar=val

**三种执行方式**
1. 以字符串的形式导入模块
```python
temp = 'sys'
model = __import__(temp)
print model.path
 
# 结果：
['/Users/study/py01']
```
2. 以字符串的形式执行函数
```python
temp = "mymodel"  #模块名称
func = "myfunc"   #函数名称
model = __import__(temp)
func1 = getattr(model, func) #到mymodel模块中找myfunc函数，如果有返回function
func1(*args, **kwargs)    #执行func1函数，即执行模块中的函数
```
3. 以字符串的形式判断类里面有没有对应的方法，根据输入的内容，执行类中对应的方法
```python
class MyClass(object):
    def sayhi(self):
        print "sayhi"
    def info(self):
        print "info"
    def do(self):
        print "do"
    def run(self):
        print "run"
def outside():
    print "outside other funciton"
 
m = MyClass()
user_input  = raw_input('Pls input function:')
if hasattr(m, user_input): #判断有没有该方法
    func = getattr(m, user_input) #获得该方法
    func()  #执行方法
else:
    print "Error!"
    setattr(m, user_input, outside) #设置方法
    func = getattr(m,user_input)
    func()
```


