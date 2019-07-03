---
title: Django 测试
date: 2017-03-28 20:55:48
tags:
  - TestCase
  - unittest
categories:
  - Django
---

#### 说明
Python单元测试类扩展了unittest测试用例,Django提供了这个基类的一些扩展，如下图：

![](https://ttxsgoto.github.io/img/django/django_test01.png)

**SimpleTestCase**:扩展了unittest.TestCase的一些基本功能
- 保存和恢复python警告机制状态
- 使用client Client
- 自定义测试时间URL maps
- 使用modified settings运行测试的能力

**TransactionTestCase**:事务测试类，继承SimpleTestCase

**TestCase**:用来测试网站正常转换unittest.TestCase到Django TestCase
- 自动加载fixtures
- 将测试包含在两个嵌套的atomic块中：一个用于整个类，一个用于每个测试
- 创建一个TestClient实例
- Django特定的断言用于测试重定向和形式错误

**LiveServerTestCase**:基本上与TransactionTestCase相同，具有一个额外的功能：它在设置的后台启动一个活动的Django服务器，并在卸载时将其关闭

#### 特殊方法
- setUp():每个测试函数运行前执行
- tearDown():每个测试函数运行完成后执行
- setUpClass(cls):必须使用@classmethod装饰器，所有test函数运行前执行一次
- tearDownClass(cls):必须使用@classmethod装饰器，所有test函数运行完成后执行一次


**默认测试客户端**
django.test.*TestCase实例中的每个测试用例都可以访问Django测试客户端的实例。此客户端可以作为self.client访问。每个测试都重新创建此客户端，因此您不必担心从一个测试到另一个测试的状态（例如Cookie）
实例如下：
```python
import unittest
from django.test import Client
 
class SimpleTest(unittest.TestCase):
    def test_details(self):
        client = Client()
        response = client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)
 
    def test_index(self):
        client = Client()
        response = client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)
 
################等同于如下################
from django.test import TestCase
 
class SimpleTest(TestCase):
    def test_details(self):
        response = self.client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)
 
    def test_index(self):
        response = self.client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)
```

#### 测试客户端
```
>>> from django.test import Client
>>> c = Client()
>>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
>>> response.status_code
200
>>> response = c.get('/customer/details/')
>>> response.content    # 返回数据主体
```

#### 测试响应属性
```
- client:用于生成导致响应的请求的测试客户端
- content:响应的主体
- context:用于呈现产生响应内容的模板的模板Context实例
- request:响应的请求数据
- wsgi_request:由生成响应的测试处理程序生成的WSGIRequest实例
- status_code:响应http状态码
- templates:用于渲染最终内容的Template实例列表，按渲染顺序排列
- resolver_match:响应的实例ResolverMatch
```

#### 常用断言（runtests/case.py）
- self.assertEqual(first, second)
- self.assertNotEqual(first, second)
- self.assertFalse(expr)
- self.assertTrue(expr)
- self.assertEqual(first, second)
- self.assertNotEqual(first, second)
- self.assertSequenceEqual(seq1, seq2)
- self.assertListEqual(list1, list2):
- self.assertTupleEqual(tuple1, tuple2):
- self.assertDictEqual(dic1, dic2):
- self.assertSetEqual(set1, set2):
- self.assertIn(mem, container):
- self.assertIs(expr1, expr2)
- self.assertIsNotNone(obj):
- self.assertIsNone(obj)
- self.assertIsInstance(obj, cls)

#### 跳过测试
unittest库提供@skipIf和@skipUnless装饰器，如果提前知道这些测试在某些条件下会失败，可以跳过测试


#### 运行
```
python manage.py test runtests.test_user
python manage.py test xxx.test		#执行xxx项目下的testx里的测试
python manage.py test animals.tests.AnimalTestCase	#单独执行某个test case
```

#### 测试数据库
测试需要数据库，django会为测试单独生成数据库。不管你的测试是否通过,当你所有的测试都执行过后,这个测试数据库就会被销毁

默认情况下,测试数据库的名字是test_DATABASE_NAME,DATABASE_NAME是你在settings.py里配置的数据库名.如果 你需要给测试数据库一个其他的名字,在settings.py中指定TEST_DATABASE_NAME的值。使用sqlite3时，数据库是在内存中创建的

#### 参考链接
https://docs.djangoproject.com/en/1.10/topics/testing/tools/
http://python.usyiyi.cn/django/topics/testing/tools.html
http://www.cnblogs.com/linxiyue/p/3886035.html
http://www.weiguda.com/blog/31/



