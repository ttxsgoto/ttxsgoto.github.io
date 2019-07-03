---
title: Django 自定义认证字段
date: 2018-02-05 22:21:34
tags:
  - Auth
categories:
  - Django
---
#### 背景
Django默认使用username字段来进行认证，而现在对于多数平台来说，更多的通过email或手机号来进行认证，自定义认证字段就是解决这类问题。

#### 说明
Django中在底层，维护着"authentication backends"列表，当调用django.contrib.auth.authenticate() 时，会尝试所有的使用通过settings通过AUTHENTICATION_BACKENDS设置的backend来进行认证，默认为`django.contrib.auth.backends.ModelBackend`，依次验证，如果匹配成功，则停止后续处理，如果后台引发PermissionDenied异常，认证失败且不会检查后面的认证。


#### 实例
通过用户名或者email进行认证

```python
from __future__ import unicode_literals
from django.contrib.auth import get_user_model
 
UserModel = get_user_model()
 
 
class CustomizedBackend(object):
    def authenticate(self, request, **kwargs):
        if 'username' in kwargs:
            auth = {'username': kwargs.get('username', '')}
        elif 'email' in kwargs:
            auth = {'email': kwargs.get('email', '')}
        else:
            auth = {}
        password = kwargs.get('password', '')
        try:
            user = UserModel._default_manager.get(**auth)
        except UserModel.DoesNotExist:
            return
        else:
            if user.check_password(password):
                return user
 
    def get_user(self, user_id):
        try:
            return UserModel._default_manager.get(pk=user_id)
        except UserModel.DoesNotExist:
            return None
```

#### settings.py
```python
# 默认登录使用的backend
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',  # 默认backend
    'apps.auth.backends.CustomizedBackend',        # 各backend依次进行验证, 直到某一个验证通过
)
```


