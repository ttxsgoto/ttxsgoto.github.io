---
title: Python Logging模块
date: 2017-03-06 21:42:55
tags:
  - Logging
categories:
  - python
---
logging为python模块提供状态、错误、信息输出的标准接口。
日志级别大小关系为：CRITICAL > ERROR > WARNING > INFO > DEBUG > NOTSET


logging.basicConfig函数各参数说明：
filename: 指定日志文件名
filemode: 和file函数意义相同，指定日志文件的打开模式，'w'或'a'
format: 指定输出的格式和内容：

    %(levelno)s: 打印日志级别的数值
    %(levelname)s: 打印日志级别名称
    %(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
    %(filename)s: 打印当前执行程序名
    %(funcName)s: 打印日志的当前函数
    %(lineno)d: 打印日志的当前行号
    %(asctime)s: 打印日志的时间
    %(thread)d: 打印线程ID
    %(threadName)s: 打印线程名称
    %(process)d: 打印进程ID
    %(message)s: 打印日志信息

datefmt: 指定时间格式，同time.strftime()
level: 设置日志级别，默认为logging.WARNING

stream: 指定将日志的输出流，可以指定输出到sys.stderr,sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略


记录日志信息
```python
#!/usr/bin/env python
#coding: utf-8
import logging

logname = 'log.log'
logging.basicConfig(
    level = logging.INFO,    #定义记录大于或等于日志级别
    format='[%(levelname)s] [%(asctime)s] --- %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename=logname,
    filemode='a',)
logging.warning("log")
```
将日志输出到文件，同时打印匹配的级别到屏幕上
```python
#!/usr/bin/env python
#coding: utf-8
import logging
logname = 'log.log'
logging.basicConfig(
    level = logging.INFO,    #定义记录大于或等于日志级别
    format='[%(levelname)s] [%(asctime)s] --- %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename=logname,
    filemode='a',)
console = logging.StreamHandler()
console.setLevel(logging.INFO)    #定义需要显示大于或等于日志级别
formatter = logging.Formatter('[%(levelname)s] [%(asctime)s]-- %(message)s')
console.setFormatter(formatter)
logging.getLogger('').addHandler(console)
logging.warning('log info')
```