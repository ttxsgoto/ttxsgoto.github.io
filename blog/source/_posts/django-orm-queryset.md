---
title: Django ORM数据查询操作优化
date: 2018-04-13 22:47:47
tags:
  - ORM
  - Queryset
categories:
  - Django
---
最近几天研究了一下django ORM查询优化，响应时间慢问题，主要原因还是代码规范和方法使用不当，如果正确使用相应方法，大部分的性能问题都是可以解决，记录如下：

#### Django QuerySet懒执行
只有访问到对应的数据时，才会访问数据库，如果再次读取查询到的数据时，不会触发访问数据库操作，返回的是QuerySet、ValuesQuerySet、ValuesListQuerySet、Model实例
会执行数据库操作的操作有：
- Iteration，对Queryset进行迭代操作
- slicing分片， 如queryset[:5]
- 序列化Pickling
- repr()/str()将对象转为字符串
- len()/list()/bool()/print()操作

```python
######## 实例一 ########
from django.db import connection
users = User.objects.all()
users = User.objects.all().select_related('group')
for user in users:
    print(user.name) # 这里只查询一次数据库
    print(user.group.name) # 这里的group为外键，查询次数依次增加, 可通过select_related()来解决多次查询数据库问题
    l = connection.queries
    print(len(l), l)
 
 
######## 实例二 ########
# 这时不会访问数据库
users = User.objects.filter(age=25)
 
# 这里需要访问数据，执行数据库查询操作
list(users) or if users: pass
 
# 再次读取数据，不会访问数据库
user = users[0]
```
#### 使用select_related 提升关联外键/一对一关联查询
Model中一般会定义外键关联，查询如果编写不当，会多次访问数据库查询，影响效率；通过select_related方法来查询外键(ForeignKey)或一对一(OneToOneField)关系，其实就是sql语句中join操作；在后面使用外键关系查询时将不需要执行数据库查询

使用prefetch_related提升关联多对多或多对一查询
prefetch_related 执行一个单独的查找，它允许预先读取多对多和多对一的对象数据，这是 select_related 做不到的。另外 perfetch_related 也可以与通用外键和关系一起使用
```python
######## 实例一 ########
DeviceInfo.objects.filter(operator_id='dc8b37483b27402d90a5a28d13ce330c')
# 对应sql如下：
SELECT `device_info`.`id`, `device_info`.`number`, `device_info`.`type`, `device_info`.`plate_no_id`, `device_info`.`status`, `device_info`.`offline_date`,  `device_info`.`operator_id` FROM `device_info` WHERE `device_info`.`operator_id` = dc8b37483b27402d90a5a28d13ce330c
DeviceInfo.objects.select_related('operator').filter(operator_id='dc8b37483b27402d90a5a28d13ce330c')
# 对应sql如下：
SELECT `device_info`.`id`,  `device_info`.`number`, `device_info`.`type`, `device_info`.`plate_no_id`, `device_info`.`status`, `device_info`.`offline_date`, `device_info`.`operator_id`, `user`.`password`, `user`.`last_login`, `user`.`is_superuser`, `user`.`id`, `user`.`created_at`, `user`.`is_deleted`, `user`.`mobile_number`, `user`.`is_active`, `user`.`is_staff`, `user`.`is_driver`, `user`.`depgroup_id` FROM `device_info` INNER JOIN `user` ON (`device_info`.`operator_id` = `user`.`id`) WHERE `device_info`.`operator_id` = dc8b37483b27402d90a5a28d13ce330c
 
######## 实例二 ########
from django.db import models
 
class City(models.Model):
    # ...
    pass
 
class Person(models.Model):
    # ...
    hometown = models.ForeignKey(City)
 
class Book(models.Model):
    # ...
    author = models.ForeignKey(Person)
 
b = Book.objects.select_related('author__hometown').get(id=4)
p = b.author         # Doesn't hit the database.
c = p.hometown       # Doesn't hit the database.
 
b = Book.objects.get(id=4) # No select_related() in this example.
p = b.author         # Hits the database.
c = p.hometown       # Hits the database.
```
#### 不要查询不需要的值
- 通过values和values_list来限制返回值
- 通过only指定字段和defer排除字段
- 如果只需要id，可以使用queryset.values_list('id', flat=True)

#### 直接使用外键值
如果只想获取外键id，可通过obj_id的方式获取，优先于obj.id;obj.id方式会为子表内容保存额外查询

#### 用count()代替len(), exists()代替if queryset
len()方法相当于会把整个queryset遍历一次，把所有的数据都取出来对象化，消耗大量的资源
#### 对缓存的queryset只进行一次遍历，使用iterator()
```python
# 如此操作可减少数据载入内存中，同时和values一起使用可大大减少内存的使用
for user in User.objects.all().iterator():
    do_something(user)
```
#### 避免多次查询
筛选表中不同条件的数据时，一般采用写多个查询进行筛选，数据多时严重影响性能
```python
users = [
'ttxsgoto01',
'ttxsgoto02',
'ttxsgoto03',
]
from models import User
######## 实例一 ########
# 这里会进行多次数据库查询操作
for user in users:
    user1 = User.objects.filter(username=user, age=21)
    user2 = User.objects.filter(username=user, age=22)
    user3 = User.objects.filter(username=user, age=23)
    user4 = User.objects.filter(username=user, sex='M')
    print(user1.count(), user2.count(), user3.count(), user4.count())
 
######## 实例二 ########
# 减少数据库查询，一次把数据查询出来
for user in users:
    _user = User.objects.filter(username=user).values_list('age', 'sex')
    user1 = filter(lambda x:True if x[0]==21 else False, _user)
    user2 = filter(lambda x:True if x[0]==22 else False, _user)
    user3 = filter(lambda x:True if x[0]==23 else False, _user)
    user4 = filter(lambda x:True if x[1]=='M' else False, _user)
    print(user1.count(), user2.count(), user3.count(), user4.count())
# 如此操作，一个条件只执行一次数据库查询，不同于实例一中会执行多次数据库查询
```

#### 创建表索引
根据业务需求，创建对应的索引字段

#### 对于复杂的数据库查询操作，使用原生SQL实现

#### 性能分析
##### 方法一: code
```python
from django.db import connection
dbsql = connection.queries # 具体sql执行情况和耗时情况
```
##### 方法二: shell
```python
- pip install django-extensions
- INSTALLED_APPS = (
    ...
    'django_extensions',
    ...
 )
- python manage.py shell_plus --print-sql # 查看执行sql和结果
```

#### 参考文档
- https://docs.djangoproject.com/en/1.10/topics/db/optimization/




