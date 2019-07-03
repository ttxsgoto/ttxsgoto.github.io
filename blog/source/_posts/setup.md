---
title: Python Setuptools模块
date: 2017-06-14 21:51:32
tags:
  - Setuptools
categories:
  - python
---

### 说明
将日常常用模块打包成tar.gz/egg/wheel包，方便其他工程复用；主要通过setuptools这个模块完成打包工作

### 简单实例
目录结构：
```
├── README		# readme信息
├── package_demo	# 包名
│   ├── __init__.py
│   └── now_time.py	# 功能模块
└── setup.py		# setup文件
```
now_time.py
```python
#!/usr/bin/env python
#coding: utf_8
import datetime
 
def now():
    return datetime.datetime.now()
 
if __name__ == '__main__':
    now()
```
setup.py
```python
#!/usr/bin/env python
#coding: utf_8
 
from setuptools import setup, find_packages
from os import path
 
here = path.abspath(path.dirname(__file__))
 
with open(path.join(here, 'README')) as f:
    long_description = f.read()
 
install_requires=[
    'gitchangelog',
]
 
setup(
    name='Package_demo',
    version='1.0.0',
    description='setup package demo',
    long_description=long_description,
    url='https://github.com/',
    author='ttxsgoto',
    author_email='359450323@qq.com',
    license='MIT',
    classifiers=[
        'Development Status :: 4 - Beta',
        'Intended Audience :: Developers',
        'Topic :: Software Development :: Build Tools',
        'License :: OSI Approved :: MIT License',
        'Programming Language :: Python',
        'Programming Language :: Python :: 2.7',
    ],
    keywords='Package_demo',
    # packages=find_packages(
    #     where='.',
    #     exclude=['contrib', 'docs', 'tests'], # 排除某些包
    # ),
    packages=['package_demo'],
    install_requires=install_requires,
)
```
#### 常用字段
- name 项目名称
- version 项目当前的版本，1.0.0表示1.0.0版，目前还处于开发阶段
- description 包的简单描述
- long_description=long_description, 较长的描述
- url 为项目访问地址
- author 为项目开发人员
- author_email 为项目开发人员邮件
- license 为本项目遵循的授权许可
- classifiers 有很多设置，具体内容可以参考官方文档, https://pypi.python.org/pypi?%3Aaction=list_classifiers
- keywords 是本项目的关键词，理解为标签
- packages 指定包,如果很多可以使用find_packages & exclude
- install_requires 依赖包安装
- extras_require 额外的依赖包

#### 打包命令
```python
python setup.py check  # 检查
python setup.py sdist  # 打包为 .tar.gz
python setup.py bdist_egg  #  创建 Eggs包
python setup.py bdist_wheel  # 创建 Wheel包
# 生成的文件均位于 dist 目录下
```
打包后的目录结构
```
python setup.py sdist
 
├── Package_demo.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   ├── requires.txt
│   └── top_level.txt
├── README
├── dist
│   └── Package_demo-1.0.0.tar.gz
├── package_demo
│   ├── __init__.py
│   └── now_time.py
└── setup.py
```
#### 包验证
```python
pip install Package_demo-1.0.0.tar.gz
In [1]: from package_demo import now_time
 
In [2]: now_time.now()
Out[2]: datetime.datetime(2017, 6, 14, 23, 10, 52, 278942)
 
```
### 分发到PyPI
首先到 PyPI 注册一个帐号, 在邮箱内确认
之后在目录新建一个 .pypirc 文件, 写入下面内容(注意填入自己的帐号密码):
```python
[pypirc]
servers = pypi
[server-login]
username:username
password:password
```
上传
```python
python setup.py register  # 将包注册到 PyPI
python setup.py register sdist upload  # 上传
```
登录验证查看是否上传成功

### 参考链接
- https://packaging.python.org/tutorials/distributing-packages/#name
- https://github.com/pypa/sampleproject/blob/master/setup.py
- https://github.com/celery/celery/blob/master/setup.py


