---
title: Python Sqlalchemy模块
date: 2017-12-24 12:27:23
tags:
  - Sqlalchemy
categories:
  - python
---

#### 说明
项目中会使用到数据库，需要创建表结构和相关属性，因对原生sql不熟悉，使用python中ORM框架qlalchemy模块来完成建表查询操作

#### 基本用法
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
 
# echo=True是开启调试
# 连接到数据库, 返回engine实例,只有触发数据库事件,才连接数据库操作
engine = create_engine('mysql+pymysql://root:root@127.0.0.1:3307/test?charset=utf8', echo=True)
 
# 声明映射, 通常一个应用使用一个base实例，所有实体类都应该继承此类对象
Base = declarative_base()
 
# 定义表字段结构
class Address(Base):
    """
    CREATE TABLE addresses (
            id INTEGER NOT NULL AUTO_INCREMENT,
            email_address VARCHAR(30) NOT NULL,
            user_id INTEGER,
            PRIMARY KEY (id),
            FOREIGN KEY(user_id) REFERENCES ttxs_users (id)
        )
    """
    __tablename__ = 'addresses'
    
    id = Column(Integer, primary_key=True)
    email_address = Column(String(30), nullable=False)
    user_id = Column(Integer, ForeignKey('ttxs_users.id'))
    user = relationship("User", back_populates='addresses') # 通过relationship()来指明关系
 
    def __repr__(self):
        return "<Address(email_address='%s')>" % self.email_address
 
 
class User(Base):
    """
    CREATE TABLE ttxs_users (
        id INTEGER NOT NULL AUTO_INCREMENT,
        name VARCHAR(30),
        fullname VARCHAR(30),
        password VARCHAR(30),
        PRIMARY KEY (id)
    )
    """
    __tablename__ = 'ttxs_users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(30))
    password = Column(String(30))
    addresses = relationship(Address, order_by=Address.id, back_populates="user") # 通过relationship()来指明关系
 
    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" % (self.name, self.fullname, self.password)
 
    @classmethod
    def add(cls, **kwargs):
    	try:
            r = cls(**kwargs)
            session.add(r)
            session.commit()
        except:
            session.rollback()
            raise
 
 
# 将表结构写入数据库中
Base.metadata.create_all(engine)
 
# 插入数据
ed_user = User(name='ttxsgoto', fullname='ttxsgoto', password='password')
print(ed_user.name, ed_user.id) # ttxsgoto None
 
ed_user.password = 'f8s7ccs'
 
# 添加ed_user的email_address
ed_user.addresses = [
    Address(email_address='ttxsgoto01@163.com'),
    Address(email_address='ttxsgoto02@163.com')
]
 
# 创建会话, Session是真正与数据库通信的handler
# 绑定数据库,方法一
Session = sessionmaker(bind=engine)
# 绑定数据库,方法二
# Session = sessionmaker()
# Session.configure(bind=engine)
 
session = Session()
 
# 插入数据
session.add(ed_user)
# 插入多条数据
session.add_all([
    User(name='ttxsgoto01', fullname='ttxsgoto01', password='ttxsgoto01'),
    User(name='ttxsgoto02', fullname='ttxsgoto02', password='ttxsgoto02'),
    User(name='ttxsgoto03', fullname='ttxsgoto03', password='ttxsgoto03')
])
 
# 提交写入数据
session.flush()
session.commit()
 
our_user = session.query(User).filter_by(name='ttxsgoto').first()
print(ed_user is our_user)
print(our_user)
print(session.dirty)
print(session.new)
 
# 查询
for instance in session.query(User).order_by(User.id):
    print(instance.name, instance.fullname, instance.password)
 
############################## 相关查询 ##############################
# ==
querys = session.query(User).filter(User.name == 'ttxsgoto')   #得到的是查询sql语句
# !=
querys = session.query(User).filter(User.name != 'ttxsgoto')
# like
querys = session.query(User).filter(User.name.like('%ttxsgoto%'))
# ilike
querys = session.query(User).filter(User.name.ilike('%ttxsgoto%'))
# in
querys = session.query(User).filter(User.name.in_(['ttxsgoto01', 'ttxsgoto02']))
# not in
querys = session.query(User).filter(~User.name.in_(['ttxsgoto01', 'ttxsgoto02']))
# is null
querys = session.query(User).filter(User.name == None)
# is not null
querys = session.query(User).filter(User.name != None)
# and
querys = session.query(User).filter(User.name == 'ttxsgoto', User.password == 'f8s7ccs')
# or
from  sqlalchemy import or_
querys = session.query(User).filter(or_(User.name == 'ttxsgoto', User.name == 'ttxsgoto01'))
# match
querys = session.query(User).filter(User.name.match('ttxsgoto'))
# all()
querys = session.query(User).all()
# first()
querys = session.query(User).first()
print(querys.name)
# one() / one_or_none()
querys = session.query(User.id == 13).one()
# count
querys = session.query(User).filter(User.name.like('%ttxsgoto%')).count()
for query in querys:
    print('query--->', query.name)
# Querying with joins
querys =session.query(User, Address).filter(User.id==Address.user_id).filter(Address.email_address=='ttxsgoto01@163.com').all()
# join
querys =session.query(User).join(Address).filter(Address.email_address=='ttxsgoto01@163.com').all() # 这里查询的是User表中信息
querys = session.query(User).join(Address, User.id == Address.user_id).all()
for u, a in querys:
    print('query--->', u.name, a.email_address)
 
# delete
user = session.query(User).filter_by(id=6).first()
session.delete(user)
session.commit()
```
#### 多表关系

##### 一对多(one to many)

一对多(one to many) Foreignkey 始终定义在多的一方, 如果关联属性(relationship)定义在一的一方为一对多

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
 
DB_URL = 'sqlite:///test.db'
engine = create_engine(DB_URL)
Base = declarative_base()
 
 
class Parent(Base):
    __tablename__ = 'parent'
 
    id = Column(Integer, primary_key=True)
    children = relationship('Child', backref='parent')  # backref建立双向关系
 
 
class Child(Base):
    __tablename__ = 'child'
 
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
 
 
# 将表结构写入数据库中
Base.metadata.create_all(engine)
```
##### 多对一(many to one)
多对一(many to one) Foreignkey 始终定义在多的一方, 如果关联属性(relationship)定义在多的一方为多对一
一个child可能有多个parent(父亲和母亲),这里的外键(child_id)和relationship(child)都定义在多(parent)的一方
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
 
DB_URL = 'sqlite:///test.db'
engine = create_engine(DB_URL)
Base = declarative_base()
 
class Parent1(Base):
    __tablename__ = 'parent1'
 
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child1.id'))
    child = relationship('Child1', backref='parent1', cascade= 'all')   # cascade设置关联删除
 
 
class Child1(Base):
    __tablename__ = 'child1'
 
    id = Column(Integer, primary_key=True)
 
# 将表结构写入数据库中
Base.metadata.create_all(engine)
```
##### 一对一(one to one)
只需在一对多的relationship加上一个参数uselist=False,或者在多对一的backref中添加uselist=False ,即将对应关系变成一对一

##### 多对多(many to many)
需要定义一张中间关联表来完成
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
 
DB_URL = 'sqlite:///test.db'
engine = create_engine(DB_URL)
Base = declarative_base()
 
class Association(Base):
    __tablename__ = 'association'
 
    left_id = Column(Integer, ForeignKey('left.id'), primary_key=True)
    right_id = Column(Integer, ForeignKey('right.id'), primary_key=True)
    extra_data = Column(String(50))
    child = relationship("Child2")
 
 
class Parent2(Base):
    __tablename__ = 'left'
 
    id = Column(Integer, primary_key=True)
    children = relationship(
        "Association",)
 
 
class Child2(Base):
    __tablename__ = 'right'
 
    id = Column(Integer, primary_key=True)
 
# 将表结构写入数据库中
Base.metadata.create_all(engine)
```

#### 使用Alembic 数据库迁移
alembic是sqlalchemy的作者开发的，用来做ORM模型与数据库的迁移与映射。
```
alembic.ini 提供了一些基本的配置
env.py 每次执行Alembic都会加载这个模块，主要提供项目Sqlalchemy Model 的连接
script.py.mako 迁移脚本生成模版
versions 存放生成的迁移脚本目录
 
alembic init   dir
# alembic revision -m "create account table"  生成Migration文件
alembic revision --autogenerate       自动生成迁移脚本
alembic revision --autogenerate -m "add user table" # 生成alembic升级脚本
alembic upgrade head          
```
使用步骤：
1. 定义好模型
2. 使用alembic创建一个仓库：`alembic init [仓库的名字，推荐使用alembic]`
3. 修改配置文件：
    - 在`alembic.ini`中，给`sqlalchemy.url`设置数据库的连接方式。这个连接方式跟sqlalchemy的方式一样的
    - 在`alembic/env.py`中的`target_metadata`设置模型的`Base.metadata`。但是要导入`models`，需要将models所在的路径添加到这个文件中
4. 将ORM模型生成迁移脚本：`alembic revision --autogenerate -m 'message'`
5. 将生成的脚本映射到数据库中：`alembic upgrade head`
6. 以后如果修改了模型，重复4、5步骤











