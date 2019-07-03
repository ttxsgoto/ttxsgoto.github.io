---
title: Python Fabric模块
date: 2017-03-19 20:47:15
tags:
  - Fabric
categories:
  - python
---
#### 说明
基于paramiko的封装，远程执行命令，方便简单，实用性强。

#### 参数
```
-l  显示可用的task
-f  指定入口文件，默认为fabfile.py
-H  指定目标主机，主机之间用，号分隔
-P  已并行执行，默认为串行
-R  指定role角色
-t  连接超时时间(s)
-T  执行命令超时时间(s)
```

#### API
```
from fabric.api import env
########## 属性 ##########
env.hosts    -主机ip
env.port    -主机端口，默认为22
env.passworkd   -ssh密码
env.roledefs    -角色分组，env.roledefs={'web1':['192.168.0.192']}，调用@roles('web1')
env.passwords   -字典，为每台机器设置密码，key是ip，value是密码，如{'root@192.168.0.200:22':'root'}，调用：@hosts('root@192.168.0.200:22')
env.exclude_hosts   -指定排除主机列表，在fab执行时，忽略列表中的机器；env.exclude_hosts= ['10.1.1.2']
 
########## API ##########
local('pwd')    -执行本地命令
lcd('/tmp')     -切换本地目录
cd('/tmp')      -切换远程目录
run('uname -s') -执行远程命令
sudo('service sshd restart')   -执行远程sudo，
put('/local/test','/remote/test')   -上传
get('/remote/test/','/local/test')  -下载
prompt  -获得用户输入信息,如:prompt('please Input password:');
confirm -获得提示信息确认,如:confirm("Continue[Y/N]？");
@task(alias='')   -函数修饰符,标识的函数为fab可调用的,非标记对fab不可见,alias设置别名
@parallel(pool_size=5)   -并行执行任务影响的最小单位是任务，所以功能启用或禁用以任务为单位,pool_size指定并发个数
@serial     -顺序执行，非并发执行
@runs_once  -只有第一台执行此函数
with cd('/root'):   pass    -with让后面的执行继承当前所在状态
 
__all__ = ['run1','hello']  -定义全局的可用tasks列表

from fabric.colors import * #用于打印显示颜色   -print(yellow("This text is yellow!",bold=True))    bold：用于设置粗体
```
#### 实例
```python
#!/usr/bin/env python
#coding: utf_8
 
from __future__ import with_statement
from __future__ import unicode_literals
 
import time
from fabric.api import local, cd, run, settings,env,task
from fabric.context_managers import prefix
from fabric.contrib.console import confirm
from fabric.decorators import roles, parallel, serial, runs_once
from fabric.colors import *
from fabric.operations import prompt
 
env.hosts=['192.168.0.192','192.168.0.200']
env.password='root'
env.user='root'
#env.roledefs={'web1':['192.168.0.192'],'web2':['192.168.0.200']}
 
__all__ = ['local_deploy','run1','hello','test_confirm','test_prompt']     #定义全局可用的tasks列表
 
@task
def hello(name):
    '''
    传递参数: fab hello:name=ttxsgoto
    '''
    print "hello,{}".format(name)
 
@task
def local_deploy():
    '''
    执行本地命令
    :return:
    '''
    with settings(warn_only=True):
        return local('ls /Users/study/studypy/train/fabric_demo/fabfile.py')
    # local('cat fabfile.py')
 
@task   #标识为fab可调用
@roles('web2')  #调用roles
def with_deploy():
    '''
    执行with函数
    :return:
    lcd 本地执行,cd 远程执行
    '''
    dir = '/etc/network/'
    with cd(dir):
        run('ls .')
 
@task
def dir():
    """
    remote list
    :return:
    """
    dir = '/etc/'
    with cd(dir):
        run('ls -l rc.local')
 
@task(default=True)
@parallel(pool_size=3)
def run1():
    """
    启动
    :return:
    """
    print time.ctime()
    hello('ttxsgoto')
    local_deploy()
    dir()
    time.sleep(2)
    print time.ctime()
    print(yellow("This text is green!",bold=True)) #用于打印显示颜色, bold：用于设置粗体
 
@task()
def runserver():
    with prefix('workon ownserver'):
        run('cd /date/ownserver && python manage.py runserver 0.0.0.0:8000')
 
@task()
def test_confirm():
    '''
    测试交互确认
    :return:
    '''
    INFO = confirm('Are you sure?[yes/no]?')
    if INFO:
        print 'yes'
    else:
        print 'no'
 
@task()
def test_prompt():
    '''
    测试输入信息
    :return:
    '''
    Text = prompt('Input word:')
    print '-----',Text
```