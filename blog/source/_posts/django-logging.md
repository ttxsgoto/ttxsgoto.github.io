---
title: Django Logging
date: 2017-11-13 22:41:28
tags:
  - Logging
categories:
  - Django
---

#### logging组成
- 记录器(Logger)
    
    logger决定消息需要处理，即将传递该消息给一个Handler
- 处理程序(Handler)
    
    handler决定如何处理logger中的每条信息，表示详细的日志行为，如写入文件或者屏幕显示等
    
    handler中也有日志级别，如果消息的日志级别小于handler的级别，handler将忽略该消息

- 过滤器(Filters)

    用于对从logger传递给handler的日志记录进行额外的控制
    
    filters可以用于修改将要处理的日志记录的优先级
    
    filters可以安装在logger或者handler上，多个filter可以串联起来实现多层filter行为
    
- 格式化(Formatter)

    日志记录需要转换成文本

#### 日志级别
- DEBUG: 用于调试底层系统信息
- INFO: 普通的系统信息
- WARNING: 警告信息
- ERROR： 错误信息
- CRITICAL: 严重错误信息

#### 使用logging

```python
import logging
 
# 调用获取logger的实例
logger = logging.getLogger(__name__)
 
# 使用
logger.error('Something wrong!')
logger.debug('debug')
logger.info('info')
logger.warning('warning')
logger.critical('critical')
logger.log('log')   # 打印消息时手动指定日志级别
logger.exception() # 创建一个error级别日志消息
```

#### 实例(settings.py)
```python 
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,   # 默认配置中的所有logger 都将禁用
    'formatters': {
       'standard': {
            'format': '%(asctime)s [%(threadName)s:%(thread)d] [%(name)s:%(lineno)d] [%(module)s:%(funcName)s] [%(levelname)s]- %(message)s'},  #日志格式
        },
 
    # 'filters': {
    #     'require_debug_false': {
    #         '()': 'django.utils.log.RequireDebugFalse',
    #     },
    # },
    'handlers': {
        'default': {
            'level':'DEBUG',
            'class':'logging.handlers.RotatingFileHandler',
            'filename': "../logs/server.log",   # 日志输出文件
            'maxBytes': 1024*1024*5,                                    # 文件大小
            'backupCount': 5,                                           # 备份份数
            'formatter':'standard',                                     #使用哪种formatters日志格式
        },
        'error': {
            'level':'ERROR',
            'class':'logging.handlers.RotatingFileHandler',
            'filename': "../logs/server.log",
            'maxBytes':1024*1024*5,
            'backupCount': 5,
            'formatter':'standard',
        },
        'console':{
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'standard'
        },
    },
    'loggers': {
        'django': {
            'handlers': ['default', 'console'],
            'level': 'DEBUG',
            'propagate': True
        },
        'django.request': {
            'handlers': ['console', 'default'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'django.db.backends': {
            'handlers': ['console',],
            'level': 'DEBUG',
            'propagate': False,
        },
        'ttxs': {
            'handlers': ['console', 'default', 'error'],
            'level': 'DEBUG',
        },
    }
```


