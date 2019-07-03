---
title: Django+Celery 计划任务
date: 2018-10-24 22:38:48
tags:
  - Djcelery
categories:
  - Django
---

#### 背景
需求为工作流审批，到达某状态后，超过一段时间没有审批，则流程自动审批到下一个状态

#### 方案
想到使用计划任务来自动流转流程，因为项目本身是Django，这里使用djcelery来完成

#### 实例

项目依赖
```python
pip install django-celery
pip install flower
```

settings.py配置
```python
INSTALLED_APPS = (
   ...
   'djcelery',
)
 
import djcelery
 
djcelery.setup_loader()
BROKER_URL = 'redis://127.0.0.1:6379/1'
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
# CELERY_RESULT_BACKEND = 'djcelery.backends.database:DatabaseBackend'    # 结果存储，存储到数据库
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/1'
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Shanghai'
 
CELERY_TASK_RESULT_EXPIRES = 3600  # celery任务执行结果的超时时间，
CELERYD_FORCE_EXECV = True  # 有些情况下可以防止死锁
CELERYD_PREFETCH_MULTIPLIER = 1
CELERYD_MAX_TASKS_PER_CHILD = 100   # 每个worker最多执行万100个任务就会被销毁，可防止内存泄露
# CELERYD_CONCURRENCY = 10  # celery worker的并发数 也是命令行-c指定的数目,事实上实践发现并不是worker也多越好,保证任务不堆积,加上一定新增任务的预留就可以
```

接下来在settings.py的同级目录中新建 celery.py文件

```python
from __future__ import absolute_import, unicode_literals
import os
 
from celery import Celery
from django.conf import settings
 
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Server.settings')
 
app = Celery('Server') #, backend='redis', broker='redis://127.0.0.1:6379/1')
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
```
修改__init__.py添加
```python
from __future__ import absolute_import, unicode_literals
  
from .celery import app as celery_app
```
在对应的app中添加tasks.py文件(这里必须为tasks.py文件，不然在admin中添加任务时找不到task)
```python
from __future__ import absolute_import, unicode_literals
from celery import shared_task, task
  
@shared_task
def auto_audit():
    print('func auto audit')
 
@task()
def add(x, y):
    print('x + y = ', x + y)
    return x + y
```
在views.py中调用
```python
from .tasks import add
  
add.delay(3, 5) #发送消息，触发后台任务
```
celery启动，当然实际环境中应该使用supervisor启动
```python
python manage.py celery worker -l info  # 启动 worker
python manage.py celery beat -l info    # 启动 beat
python manage.py celery worker -B -l info  # 启动 worker 和 beat
python manage.py celery flower --address=127.0.0.1 --port=5555 --basic_auth=admin:admin    # 启动celery flower监控
 
```

#### Admin后台添加计划任务
在DJCELERY中Periodic tasks添加计划任务即可

#### Flower监控celery
主要用于监控任务执行是否成功，和broker和worker对应的状态



