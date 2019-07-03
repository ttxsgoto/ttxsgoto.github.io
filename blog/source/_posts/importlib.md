---
title: Python Importlib模块
date: 2017-04-16 15:44:11
tags:
  - Importlib
categories:
  - python
---
通过字符串导入模块，动态导入模块，3种方式,记录如下：

方式一：
```python
os1 = __import__('os')
os1.path.join <==> from os.path import join
```
方式二：
```python
import imp
os2 = imp.load_module('os',*imp.find_module('os'))
os2.path.join <==> from os.path import join
```
方式三：
```python
module = importlib.import_module('os')
os_path =  getattr(module,'path')
os_path.join <==> from os.path import join
```

