---
title: Django Signals信号
date: 2017-09-27 22:41:10
tags:
  - Signals
categories:
  - Django
---
### 说明

Django 提供一个“信号分发器”，允许解耦的应用在框架的其它地方发生操作时会被通知到。简单来说，信号允许特定的sender通知一组receiver某些操作已经发生;这在多处代码和同一事件有关联的情况下很有用

### 预定义信号
- 在模型 save()方法调用之前或之后发送
    ```
    django.db.models.signals.pre_save 
    django.db.models.signals.post_save
    ```
- 在模型delete()方法或查询集的delete() 方法调用之前或之后发送
    ```
    django.db.models.signals.pre_delete
    django.db.models.signals.post_delete
    ```
- 模型上的 ManyToManyField 修改时发送
    ```
    django.db.models.signals.m2m_changed
    ```
- Django建立或关闭HTTP 请求时发送
    ```
    django.core.signals.request_started
    django.core.signals.request_finished
    ```

### 定义和发送信号
#### 定义信号
所有信号都是 django.dispatch.Signal 的实例。providing_args是一个列表，由信号将提供给监听者的参数名称组成。理论上是这样，但是实际上并没有任何检查来保证向监听者提供了这些参数。
 ```python
 from django.dispatch import Signal
 s_email_sended = Signal(providing_args=[
    'email_tpl', 'email_subject', 'email_content', 'email_cate', 'sender', 'position',
    'candidate', 'candidate_email', 'interviewer', 'interview_email'
])
 
# s_email_sended实例信号，向接受者提供了列表中的参数，最终接受者得到的参数还是需要看send()发送过来的参数
 ```
 
 #### 发送信号
 Django中有两种方法用于发送信号:
 
- Signal.send(sender, **kwargs)
- Signal.send_robust(sender, **kwargs)

send()  不会捕获任何由receiver 产生的异常。它会简单地让错误往上传递。所以在错误产生的情况，不是所有receiver 都会获得通知.

send_robust()捕获所有继承自Python Exception类的异常，并且确保所有receiver 都能得到信号的通知。如果发生错误，错误实例会在产生错误的receiver 的二元组中返回.

调用 Signal.send()或者Signal.send_robust()来发送信号。你必须提供sender 参数（大多数情况下它是一个类），并且可以提供尽可能多的关键字参数。
 ```python
 s_email_sended.send(
            sender=staff,
            candidate=review.resume_forward.candidate,
            candidate_email=review.resume_forward.candidate.email,
            email_subject=subject,
            email_content=message,
            email_cate=FlowEmailSendedLog.EMAIL_CATE_INTERVIEWER
        )
 ```
 
#### Receiver 函数
我们需要定义接收器函数，回调函数。接受器可以是Python函数或者方法
```python
def my_callback(sender, **kwargs):
    print("Request finished!")
# 注意函数接受一个sender参数，以及通配符关键字参数(**kwargs)；所有信号处理器都必须接受这些参数
```
#### 绑定receivers到signal

1.使用receiver() 装饰器来自动连接
```python
from django.core.signals import request_finished
from django.dispatch import receiver
 
@receiver(request_finished)
def my_callback(sender, **kwargs):
    print("Request finished!")
# 现在，我们的my_callback函数会在每次请求结束时调用
```
2.手动方式
```python
from django.core.signals import request_finished
request_finished.connect(my_callback)
```
 
 #### 断开信号
  Signal.disconnect([receiver=None, sender=None, weak=True, dispatch_uid=None])
  
  调用Signal.disconnect()来断开信号的接收器。 Signal.connect()中描述了所有参数。如果接收器成功断开，返回 True ，否则返回False。

receiver 参数表示要断开的已注册receiver。如果使用dispatch_uid 标识receiver，它可以为None
 
 
### 自定义signal
```python
# 定义信号，定义的信号应该在信号发送过来的字段
s_email_sended = Signal(providing_args=[
    'email_tpl', 'email_subject', 'email_content', 'email_cate', 'sender', 'position',
    'candidate', 'candidate_email', 'interviewer', 'interview_email'
])
 
from django.dispatch import Signal
from django.dispatch import receiver
 
# 发送信号
s_email_sended.send(
            sender=staff,
            candidate=candidate,
            candidate_email=candidate.email,
            email_subject=subject,
            email_content=message,
            email_cate=EmailSendedLog.EMAIL_CATE_TYPE
        )
 
# 接收信号
@receiver(s_email_sended)
def create_email_send_log(**kwargs):
    for field in ('sender', 'email_subject', 'email_content', 'email_cate'):
        if not kwargs.get(field):
            logs.error("{} can't be null: email_sender/email_subject/email_content/email_cate".format(field))
            return
 
    kwargs.pop('signal')
    EmailSendedLog.objects.create(**kwargs)
```



