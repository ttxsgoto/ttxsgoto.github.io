---
title: Django Models
date: 2017-04-05 20:07:24
tags:
  - Models
categories:
  - Django
---
#### 字段
```
1、models.AutoField 自增列 = int(11)
　　如果没有的话，默认会生成一个名称为 id 的列，如果要显示的自定义一个自增列，必须将给列设置为主键 primary_key=True。
2、models.CharField　　字符串字段
　　必须 max_length 参数
3、models.BooleanField　　布尔类型=tinyint(1)
　　不能为空，Blank=True
4、models.ComaSeparatedIntegerField　　用逗号分割的数字=varchar
　　继承CharField，所以必须 max_lenght 参数
5、models.DateField　　日期类型 date
　　对于参数，auto_now = True 则每次更新都会更新这个时间；auto_now_add 则只是第一次创建添加，之后的更新不再改变。
6、models.DateTimeField　　日期类型 datetime
　　同DateField的参数
7、models.Decimal　　十进制小数类型 = decimal
　　必须指定整数位max_digits和小数位decimal_places
8、models.EmailField　　字符串类型（正则表达式邮箱） =varchar
　　对字符串进行正则表达式
9、models.FloatField　　浮点类型 = double
10、models.IntegerField　　整形
11、models.BigIntegerField　　长整形
　　integer_field_ranges = {
　　　　'SmallIntegerField': (-32768, 32767),
　　　　'IntegerField': (-2147483648, 2147483647),
　　　　'BigIntegerField': (-9223372036854775808, 9223372036854775807),
　　　　'PositiveSmallIntegerField': (0, 32767),
　　　　'PositiveIntegerField': (0, 2147483647),
　　}
12、models.IPAddressField　　字符串类型（ip4正则表达式）
13、models.GenericIPAddressField　　字符串类型（ip4和ip6是可选的）
　　参数protocol可以是：both、ipv4、ipv6
　　验证时，会根据设置报错
14、models.NullBooleanField　　允许为空的布尔类型
15、models.PositiveIntegerFiel　　正Integer
16、models.PositiveSmallIntegerField　　正smallInteger
17、models.SlugField　　减号、下划线、字母、数字
18、models.SmallIntegerField　　数字
　　数据库中的字段有：tinyint、smallint、int、bigint
19、models.TextField　　字符串=longtext
20、models.TimeField　　时间 HH:MM[:ss[.uuuuuu]]
21、models.URLField　　字符串，地址正则表达式
22、models.BinaryField　　二进制<br>23、models.ImageField   图片<br>24、models.FilePathField 文件
 
更多字段：
1、null=True 数据库中字段是否可以为空,默认为False
2、blank=True 表单验证允许该字段为空，如果为blank=False，表示该字段必填
　　django的 Admin 中添加数据时是否可允许空值,True允许为空，False时，表示该字段为必填
3、primary_key = False 主键，对AutoField设置主键后，就会代替原来的自增 id 列
4、auto_now 和 auto_now_add
　　auto_now   自动创建---无论添加或修改，都是当前操作的时间
　　auto_now_add  自动创建---永远是创建时的时间
5、choices
GENDER_CHOICE = (
        (u'M', u'Male'),
        (u'F', u'Female'),
    )
  如：gender = models.CharField(max_length=2,choices = GENDER_CHOICE)
6、max_length
7、default　　设置默认值
8、verbose_name　　Admin中字段的显示名称，类似于设置别名
9、name|db_column　　数据库中的字段名称
10、unique=True　　不允许重复，整表唯一
11、db_index = True　　数据库索引
12、editable=True　　在Admin里是否可编辑
13、error_messages=None　　错误提示
14、auto_created=False　　自动创建
15、help_text　　在Admin中提示帮助信息，即使字段不在表单中使用，它对生成文档也很用
16、validators=[]
17、upload-to
18、db_table='xxx' 重新设置表名称
19、related_name='xxxx'  是将外键中的 "topping_set" (取自 类 的名字), 设置为自定义的对象集描述符，一般用于当一个对象要被另一个对象关联不止一次时使用，这个参数才真正有用，如下：
    models.ForeignKey(Category, related_name="primary_storys")
    models.ForeignKey(Category, related_name="secondary_storys")
20、related_query_name='xxxx'    用于目标模型的反向过滤
 
元选项（非必须）
class Meta:
        verbose_name = u'企业/组织'          #别名
        verbose_name_plural = u'企业/组织'   #复数别名
        db_table = 'organs_organ'          #数据库表名
        ordering = ('codename', )          #排序
字段说明链接：
http://python.usyiyi.cn/translate/django_182/ref/models/fields.html#common-model-field-options
```

#### 数据库表关系
- 一对多，models.ForeignKey(A)
	当一张表中创建一行数据时，有一个单选的下拉框（可以被重复选择）
- 一对一，models.OneToOneField(B)
	在某表中创建一行数据时，有一个单选的下拉框（下拉框中的内容被用过一次就消失）
- 多对多，authors = models.ManyToManyField(C)
	在某表中创建一行数据是，有一个可以多选的下拉框

#### 模型的属性
objects 模型最重要的属性是Manager。它是Django 模型进行数据库查询操作的接口，并用于从数据库获取实例。如果没有自定义Manager，则默认的名称为objects。Managers 只能通过模型类访问，而不能通过模型实例访问；也可以自定义方法；
```python
# models.py
class User(models.Model):
    username = models.CharField(u'用户名', max_length=255)
    email = models.EmailField(u'Email')
    is_active = models.BooleanField(u'是否激活', default=True)
 
    objects = UserManager()
 
# managers.py
class UserManager(Manager):
 
    def __init__(self):
        super(BaseManager, self).__init__()
 
    def create_staff(self, **kwargs):
        from .models import Staff
        staff = Staff(**kwargs)
        staff.save()
        return staff
# 调用
User.objects.create_staff(**kwargs)
```

#### 模型的方法
可以在模型上定义自定义的方法来给你的对象添加自定义的“底层”功能。Manager 方法用于“表范围”的事务，模型的方法应该着眼于特定的模型实例。

管理器方法可以通过self.model 来得到它所属的模型类
```python
# models.py
class User(models.Model):
    username = models.CharField(u'用户名', max_length=255)
    email = models.EmailField(u'Email')
    is_active = models.BooleanField(u'是否激活', default=True)
 
    def is_true(self):
        return True if self.is_active else False
 
# views.py	仅仅为了说明，下面例子没有任何意义
user = User.objects.get(id=id)
if user.is_true():
    return 'right'
    
```

#### get()/filter()区别
- models.objects.get() 获取到的是一个对象，如果没有抛出DoesNotExist异常；
- get()得到的对象，更新需要obj.name=name, obj.save()
- models.objects.filter()数据过滤，得到是一个查询集-对象列表，如果没有返回[]
- filter()得到的对象，更新时直接models.objects.filter().update()

#### model.object.create()/model.save()区别
```python
Account.objects.create(**kwargs) #调用save()方法，保存到数据库，返回实例object
Account(**kwargs)    		 #为类对象，没有调用save()方法，没有保存到数据库，直到调用save()方法将数据保存
```
#### 模型继承
##### 抽象基类
只想使用父类来持有一些信息，不想在每个子模型中都定义一次，这个类永远不会单独使用
编写完基类之后，在 Meta类中设置 abstract=True ，该类就不能创建任何数据表，如果抽象化基础类和它的子类有相同的项，那么将会出现error（并且Django将返回一个exception），例子如下：
```python
class BaseModel(models.Model):
    created_time = models.DateTimeField(u'创建时间', auto_now_add=True)
 
    class Meta:
        abstract = True
        ordering = ['-created_time']
 
class Department(BaseModel):
    name = models.CharField(u'部门名称', max_length=100, default='')
    contact = models.CharField(u'联系电话', max_length=20, null=True, blank=True, default='')
    desc = models.TextField(u'描述', null=True, blank=True, default='')
 
    class Meta:
        verbose_name = u'部门'
        verbose_name_plural = u'部门'
        db_table = 'department'
 
    def __unicode__(self):
        return u'%s' % self.name
```
在 ForeignKey或 ManyToManyField字段上使用 related_name属性，你必须总是为该字段指定一个唯一的反向名称

##### 多表继承
继承一个已经存在的模型且想让每个模型具有它自己的数据库表
```python
from django.db import models
 
class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)
 
class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```
##### 代理继承
只是想改变模块Python 级别的行为，而不用修改模型的字段,更改默认的manager，或者添加一个新的方法；为原始模型创建一个代理，你可以创建，删除，更新代理 model 的实例，而且所有的数据都可以像使用原始 model 一样被保存
不同之处在于：你可以在代理 model 中改变默认的排序设置和默认的 manager ，更不会对原始 model 产生影响
```python
from django.db import models
 
class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
 
class MyPerson(Person):
    class Meta:
        proxy = True
 
    def do_something(self):
        # ...
        pass
 
class OrderedPerson(Person):
    class Meta:
        ordering = ["last_name"]
        proxy = True
 
```
#### 多对多表结构中添加字段

```python
class Person(models.Model):
    name = models.CharField(max_length=128)
    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Membership(models.Model):
    person = models.ForeignKey(Person)
    group = models.ForeignKey(Group)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```
```python
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
...     date_joined=date(1962, 8, 16),
...     invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
[<Person: Ringo Starr>]
>>> ringo.group_set.all()
[<Group: The Beatles>]
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason="Wanted to form a band.")
>>> beatles.members.all()
[<Person: Ringo Starr>, <Person: Paul McCartney>]
```
#### 关联对象查询
```python
from django.db import models
 
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()
 
    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Author(models.Model):
    name = models.CharField(max_length=50)
    email = models.EmailField()
 
    def __str__(self):              # __unicode__ on Python 2
        return self.name
 
class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()
 
    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```
##### 一对多关系

正向查询
```python
>>> e = Entry.objects.get(id=2)
>>> e.blog # Returns the related Blog object.
```
反向查询
```python
# 管理器的名字为entry_set,其中entry为源模型的小写名称，当然这个名称也可以自定义，通过在ForeignKey 定义时设置related_name 参数来覆盖foo_set 的名称。例如，如果Entry 模型改成blog = ForeignKey(Blog, related_name='entries')，相对于的管理器也应该为entries
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.

# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```
##### 多对多关系

多对多关系的两端都会自动获得访问另一端的API。这些API 的工作方式与一对多关系一样
```python
e = Entry.objects.get(id=3)
e.authors.all() # Returns all Author objects for this Entry.
e.authors.count()
e.authors.filter(name__contains='John')
 
a = Author.objects.get(id=5)
a.entry_set.all() # Returns all Entry objects for this Author.
```
##### 一对一关系

正向查询
```python
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry)
    details = models.TextField()
 
ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
```
反向查询
```python
#一对一关系中的关联模型同样具有一个管理器对象，但是该管理器表示一个单一的对象而不是对象的集合；如果没有对象赋值给这个关联关系，Django 将引发一个DoesNotExist 异常；
e = Entry.objects.get(id=2)
e.entrydetail # returns the related EntryDetail object
```
#### select_for_update()
```
用于事务，返回一个for update 锁
Returns a new QuerySet instance that will select objects with a FOR UPDATE lock.
因QuerySet的延迟查询特性, copy一份用于update, 以便于不影响初始查询的candidates集
```
#### 批量创建
使用django.db.models.query.QuerySet.bulk_create()批量创建对象，减少SQL查询次数
```python
objList = [a, b, c,] #none are saved
objList = model(record=record, receiver=receiver) # 上述两种格式都是批量创建对象的数据格式
model.objects.bulk_create(objList)
```

#### 执行原始sql
在模型查询API不够用的情况下，你可以使用原始的SQL语句
```python
Person.objects.raw('SELECT id, first_name, last_name, birth_date FROM myapp_person')
```






