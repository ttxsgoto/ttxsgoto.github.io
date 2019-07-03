---
title: DRF Throtting
date: 2017-08-10 19:56:37
tags:
  - DRF
  - Throtting
categories:
  - DRF
---
Django Rest framework 频率控制配置说明
#### 全局设置
setting.py
```python
    # 设置节流方案
    'DEFAULT_THROTTLE_CLASSES': (
        # 开启匿名用户接口请求频率限制
        'rest_framework.throttling.AnonRateThrottle',
        # 开启授权用户接口请求频率限制
        'rest_framework.throttling.UserRateThrottle',
        # 开启自定义设置接口请求频率，在views中通过设置throttle_scope 来使用
        'rest_framework.throttling.ScopedRateThrottle',
        # 自定义
        'app.throttling.UserRecordThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        # 频率限制有second, minute, hour, day
        # 匿名用户请求频率
        'anon': '1000/day',
        # 授权用户请求频率
        'user': '20000/day',
        # 自定义请求频率,DEFAULT_THROTTLE_CLASSES中需要设置ScopedRateThrottle
        'ttxs': '10/minute',
        # 自定义请求频率
        'user_record': None
    },
```
#### 自定义
throttling.py
```python
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle
 
class UserRecordThrottle(UserRateThrottle):
    scope = 'user_record'
    rate = '5/minute'
```
使用方式说明：

方式一: 在views.py中使用，设置为throttle_classes = ([UserRecordThrottle,])

方式二: 在setting.py中设置，
- 在DEFAULT_THROTTLE_CLASSES添加'app.throttling.UserRecordThrottle'
- DEFAULT_THROTTLE_RATES中添加user_record
- 在views.py中通过throttle_scope = 'user_record' 调用

#### 类视图使用
```python
from rest_framework.response import Response
from rest_framework.throttling import UserRateThrottle
from rest_framework.views import APIView
 
class ExampleView(APIView):
    throttle_classes = ([UserRateThrottle,UserParserRecordThrottle])
    throttle_scope = 'ttxs' # 设置了ScopedRateThrottle对应的RATES
    
    def get(self, request, format=None):
        pass
        return Response('ok')
 
# 超过设置访问频率后，错误信息
{
  "msg": "request was throttled.",
  "code": 10429
}
```
#### 其他说明
1. 匿名用户频率如果设置大于授权用户频率，则以授权用户频率为准
2. 频率限制是针对单个接口的频率，而不是所有接口的频率
