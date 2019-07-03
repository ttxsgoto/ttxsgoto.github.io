---
title: Python ConfigParser模块
date: 2017-03-07 21:57:05
tags:
  - ConfigParser
categories:
  - python
---
ConfigParser模块记录常用方法
```python
#!/usr/bin/env python
#coding: utf-8
import ConfigParser
def main():
    """
    基本的读取配置文件
    -read(filename) 直接读取ini文件内容
    -sections() 得到所有的section，并以列表的形式返回
    -options(section) 得到该section的所有option
    -items(section) 得到该section的所有键值对
    -get(section,option) 得到section中option的值，返回为string类型
    -getint(section,option) 得到section中option的值，返回为int类型，还有相应的getboolean()和getfloat() 函数
    基本的写入配置文件
    -add_section(section) 添加一个新的section
    -set( section, option, value) 对section中的option进行设置，需要调用write将内容写入配置文件
    """
    cf = ConfigParser.ConfigParser()
    cf.read('db.txt')
    sec = cf.sections()                         #获取所有sections的值
    print sec
    opt = cf.options('db1')                     #获取指定sections的options
    print opt
    val = cf.items('db1')                       #获取指定section的配置信息，为list
    print val,type(val)
    val_str = cf.get('db1', 'db_host')          #获取sections中option的值
    print val_str
    cf.set('db1','db_host','192.168.1.55')      #设置某个option的值
    cf.write(open('db.txt','w'))
    try:
        cf.add_section('ttxsgoto')              #添加一个section
        cf.set('ttxsgoto', 'hostname', 'ttxsgoto')
        cf.write(open('db.txt','w'))
    except Exception:
        pass
    cf.remove_option('ttxsgoto', 'hostname')    #删除option
    cf.remove_section('ttxsgoto')               #删除section
    cf.write(open('db.txt','w'))

def write():
    config = ConfigParser.RawConfigParser()
    config.add_section('Section1')
    config.set('Section1','an_int','15')
    config.set('Section1','a_bool','true')
    config.set('Section1','a_float','3.1415')
    config.set('Section1', 'baz', 'fun')
    config.set('Section1', 'bar', 'Python')
    config.set('Section1','foo','%(bar)s is %(baz)s !')
    with open('example.cfg','wb') as configfile:
        config.write(configfile)

def read():
    config = ConfigParser.RawConfigParser()
    config.read('example.cfg')
    a_float = config.getfloat('Section1', 'a_float')
    an_int = config.getint('Section1', 'an_int')
    print a_float + an_int
    if config.getboolean('Section1', 'a_bool'):
        print config.get('Section1','foo')

def read1():
    config = ConfigParser.ConfigParser()
    config.read('example.cfg')
    print config.get('Section1', 'foo', 0)  #0 默认，显示定义的字符串
    print config.get('Section1','foo',1)    #设置为1，显示原字符串
    print config.get('Section1','foo',0,{'bar':'Document','baz':'evil'}) #设置section的对应的options

def read2():
    config = ConfigParser.SafeConfigParser({'bar':'Life','baz':'hard'})
    config.read('example.cfg')
    print config.get('Section1','foo')  #"Python is fun!"
    config.remove_option('Section1', 'bar')
    config.remove_option('Section1','baz')
    print config.get('Section1','foo')  #"Life is hard!"
    
if __name__ == '__main__':
    main()
```



