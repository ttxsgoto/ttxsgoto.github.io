---
title: Python Class
date: 2017-02-15 20:40:38
tags:
  - Class
categories:
  - python
---
### 相关概念
---
类：是对事物的抽象，人/动物/机器,如汽车模型；创建实例的模板
对象：是类的一个实例，如大客车；则是一个一个具体的对象，各个实例拥有的数据相互独立，互不影响
范例说明：汽车模型可以对汽车的特征和行为进行抽象，然后可以实例化为一台真实的汽车实体出来

方法：人会走，会思考，定义一个类的各个功能,类中定义的函数
数据成员：类变量或者实例变量用于处理类及其实例对象的相关的数据
消息传递：狗叫了，人听见了，就叫通信

继承：即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。例如，有这样一个设计：一个Dog类型的对象派生自Animal类，这是模拟”是一个（is-a）”关系（例图，Dog是一个Animal）,狗都四条腿走路
封装：人不能引用狗的特性，比如四条腿走路
多态性：一个叫的功能，可能是低吼，也也可能是大声叫
抽象性：简单复杂的现实问题的途径，它可以为具体问题找到最恰当的类定义
类方法重写：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写

静态字段：在class中直接定义的字段，直接通过class去获取，也可通过实例来访问，如 value = "abc"
动态字段：在class中对象中定义的字段，通过class的实例来访问，不能通过class直接访问，如在init函数后定义的字段
静态方法：在类中直接定义，没有self参数，同时使用staticmethod装饰器装饰，访问时直接通过class访问

```python
#!/usr/bin/env python
#coding:utf-8
class Person(object):
    name = "静态字段"
    def __init__(self,name,age):
        self.Name = name
        self.Age = age
    def run(self):
        print self.Name + "正在跑步"
    @staticmethod
    def run1():
        print "静态方法"
    @classmethod
    def run2(cls):
        print "类方法"
    @property    #转换为特性，直接以属性的形式访问
    def Bar(self):
        print self.Name

person1 = Person('人1',20)
# 访问动态字段
print person1.Name
# 访问动态方法
person1.run()
# 访问静态字段
print Person.name
# 访问静态方法
Person.run1()
# 调用类方法
Person.run2()
# property的访问形式,直接以属性的形式访问
person1.Bar
```
---
### 装饰器
@staticmethod ：类中的静态方法设置
@classmethod：类方法设置
@property ：将类中的方法转换为特性，直接以属性的形式访问

@staticmethod和@classmethod的作用与区别：
```
@staticmethod和@classmethod都可以直接类名.方法名(),实例名.方法名()调用
@staticmethod不需要表示自身对象的self和自身类的cls参数，和使用函数一样
@classmethod也不需要self参数，但第一个参数需要是表示自身类的cls参数
```

python类的定义：

    使用class关键字定义一个类，并且类名的首字母要大写
    当程序员需要创建的类型不能用简单类型表示时就需要创建类
    类把需要的变量和函数组合在一起，这种包含也称之为“封装”

对象的创建：
创建对象的过程称之为实例化；当一个对象被创建后，包括三个方面的特性： 对象的句柄，属性和方法
句柄：用来区分不同的对象，如a，b
对象的属性和方法与类中的成员变量和成员函数对应

---
### 类的属性
类由属性和方法组成，类的属性是对数据的封装，类的方法则是对类的行为的封装。类的属性按使用范围分为公有属性和私有属性，类的属性范围取决于属性的名称
公有属性：所谓公有属性就是在类中和类外调用的属性
私有属性：不能被类以外的函数调用（可以通过instance._classname_attribute方式访问，但只用于调试程序）
定义方式：以“\__”双下划线开始的成员变量就是私有属性，否则是公有属性
私有属性通过内部方法调用，实现对数据的封装隐藏。
内置属性：由系统在定义类的时候默认添加的，由前后两个下划线构成__dict__，__module__

---
### 类的方法
和函数定义一样，但是需要self作为第一个参数
类的方法也分为：公有方法和私有方法
    私有方法：不能被外部的类和方法调用，私有方法的定义和私有属性都是一样的，在方法的前面加上“__”双下划线
    类方法：被classmethod()函数处理过的函数，能被类所调用，也能被对象所调用（是继承的关系）
    静态方法：相当与“全局函数”，可以被类直接调用，可以被所有实例化对象共享，通过staticmethod()定义静态方法没有“self”语句；
    用于区分函数和类的方法（必须有一个self），self参数表示指向对象本身


内部类：
    所谓内部类，就是在类的内部定义的类，主要目的是为了更好的抽象现实世界;
    一般不赞同使用内部类，会使程序结构复杂，但是理解内部类有助于理解模块的调用

---
### 构造函数与析构函数
构造函数：用于初始化类的内部状态，python提供的构造函数是__init__();
__init__()方法是可选的，如果不提供，python会给出一个默认的__init__方法
__init__(self,name,score):  用于定义初始化类的属性，在实例时，可以将相关属性的值定义好
一般对数据的获取需要自定义的get和set方法

析构函数：用于释放对象占用的资源，python提供的析构函数是__del__();
__del__()也是可选的，如果不提供，则python会在后台提供默认析构函数
如果要显式的调用析够函数，可以使用del关键字，方式如下：  del  对象名

__call__()方法：通过实例化后，直接使用person1()来执行call方法

__str__()方法：通过实例化后，直接print person1 就可以显示str中返回的内容

```python
#!/usr/bin/env python
#coding:utf-8
class Person(object):
    def __init__(self,name,age):
        self.Name = name
        self.Age = age
    def run(self):
        print self.Name + "正在跑步"

    def __del__(self):
        print "解释器要销毁了"

    def __call__(self):
        print "这是call方法"
person1 = Person('人1',20)
person1()    #可以通过实例化后，添加括号直接执行call方法
person1.run()
```
---
### 类的特殊成员
```
__doc__	   类的描述信息，print Foo.__doc__
__module__ 当前操作的对象在那个模块，f = Foo();print f.__module__  # 输出__main__，即：输出模块
__class__  当前操作的对象的类是什么，f = Foo();print f.__class__   # 输出<class '__main__.Foo'>，即：输出类
__init__   构造方法，通过类创建对象时，自动触发执行
__del__	   析构方法，当对象在内存中被释放时，自动触发执行
__call__   对象后面加括号，触发执行；f = Foo(); f()或者 Foo()() 执行__call__方法
__dict__   类或对象中的所有成员;	f = Foo();print Foo.__dict__,print f.__dict__
__str__	   默认输出该方法的返回值;  f = Foo();print f
__getitem__  用于索引操作，如字典以,表示获取
__setitem__	 用于索引操作，如字典以,表示设置
__delitem__  用于索引操作，如字典以,表示删除
__iter__   用于迭代器，之所以列表、字典、元组可以进行for循环，因为类型内部定义了__iter__
__new__    在__init__之前被调用的特殊方法,用来创建对象并返回之的方法
```

---
### 垃圾回收机制
    python采用垃圾回收机制来清理不再使用的对象；python提供gc模块释放不再使用的对象python采用“引用计数”的算法方式来处理回收，即：当某个对象在其作用域内不再被其他对象引用的时候，python会自动清除对象； 
    python的函数collect()可以一次性收集所有待处理的对象（gc.collect()）

---
### 类的继承，多继承
我们定义一个class时，可以从某个现有的class继承，新的class称之为子类（Subclass），而被继承的class的class称之为父类（Base class）。

```python
#!/usr/bin/env python
#coding:utf-8
class Father(object):
    def __init__(self):
        self.Fname ="father"
    def Func(self):
        print "Father func "
    def run(self):
        print "Father func_public"
class Son(Father):
    def __init__(self):
        self.Sname = "son"
    def Bar(self):
        print "Son bar_function"
    def run(self): #重写父类的方法
        Father.run(self)
        print "xxxxxxxxxx"

s1 = Son()
s1.Func()
s1.run()
```
### 多继承
```python
class A(object):
    def __init__(self):
        print "A class"
    def run(self):
        print "THis is A run Function"
class B(A):
    def __init__(self):
        print "B class"
    def run(self):
        print "This is B run Function"
class C(A):
    def __init__(self):
        print "C class"
    def run(self):
        print "This is C run Function"
class D(B,C):
    def __init__(self):
        print "D class"

c = D()
c.run()
```

在继承关系中，如果一个实例的数据类型是某个子类，那么它的数据类型也可以看着是父类的类型；但是反过来不行
```python
#!/usr/bin/env python
#coding:utf8

class SchoolMember(object):#基类
	def __init__(self,name,age,sex):
    	self.name=name
		self.age=age
		self.sex=sex
	def tell(self):
    	print """--info of %s----
        	name:%s
			age:%s
            sex:%s
            """ %(self.name,self.name,self.age,self.sex)
class School(object):
	def __init__(self,name,addr,tel):
    	self.school_name=name
        self.addr=addr
        self.tel=tel
        self.stu_list=[]
        self.tech_list=[]
class Student(SchoolMember,School):#子类，继承，多继承
	def __init__(self,name,age,sex,grade,school):
    	SchoolMember.__init__(self,name,age,sex)#初始化基类变量
		self.grade=grade
		self.school=school
    def pay_money(self):
		print "-----%s is paying the tuition fee----" %(self.name)
	def tell(self):
		SchoolMember.tell(self)
		print "---------from school name : ---------%s" %(self.school.school_name)

school1 = School('AAAA','AAAA',999)
school2 = School('BBBB','BBBB',999)
s1 = Student('a',23,'M','python',school1)
s2 = Student('b',24,'M','Linux',school2)
s1.tell()
s2.tell()
```
---
### 多态
当子类和父类都存在相同的方法时，子类的方法会覆盖父类的方法，在代码运行时，总是会调用子类的相应方法
当父类中有某种方法，传入的任何类型只要是父类或者子类就会自动的调用相关父类或子类的方法，即多态

“开闭”原则：
对扩展开放：允许新增子类
对修改封闭：不需要修改依赖父类的相关函数

---
### 新式类和经典类
在定义时，有无object

区别：
1.在继承\__init\__函数写法
新式类：Father.\__init\__(self.name,self.sex)
新式类：super(Son,self).\__init\____(name,age,sex)

2.继承特性
经典类：深度优先
新式类：广度优先

```python
#!/usr/bin/env python
#coding:utf-8
class A(object):
    def __init__(self):
        print "A class"
    def run(self):
        print "THis is A run Function"
class B(A):
    def __init__(self):
        print "B class"
        A.__init__(self)    #新式类继承init函数，方法一
#         super(B, self).__init__() #新式类继承init函数，方法二
class C(A):
    def __init__(self):
        print "C class"
    def run(self):
        print "This is C run Function"
class D(B,C):
    def __init__(self):
        print "D class"
d = D()
d.run()
"""
经典类(深度优先)：D-B-A-C
结果：
D class
THis is A run Function
新式类(广度优先)：
结果： D-B-C-A
D class
This is C run Function
"""
```
