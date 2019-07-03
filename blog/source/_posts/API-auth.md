---
title: Python API接口认证
date: 2017-03-27 22:09:52
tags:
  - Api
categories:
  - python
---
#### 背景
之前写过一篇文章[《简单API的实现》](https://ttxsgoto.github.io/2017/03/26/API/)，这篇文章说明了通过api方式把数据传递给服务端处理，但这过程中没有认证功能，无论谁发任何内容，都一并接收，这样明显存在不安全性，这篇文章在原来的基础上，添加了接口认证功能。

#### 接口认证方式
方式一：
```
客户端：
    - 通过定义密钥
    - 将密钥加密发送给服务端
服务端：
    - 服务端定义相同的密钥
    - 通过相同的加密算法，得到一个值
    - 把服务端加密后的值和客户端发送过来的加密的密钥进行对比
缺点：
    - 密钥固定不变，加密后的值固定不变，易暴露，安全性低
```
方式二：
```
客户端：
    - 通过定义密钥
    - 将秘钥和当前时间戳一起加密，得到一个值
    - 将加密后密钥和时间戳一起发送给服务端
服务端：
    - 服务端定义相同的密钥
    - 将当前服务器端时间和发送过来的客户端时间进行对比，设置可接受的时间范围如：120s；如果服务器当前时间-发送过来的客户端>120s ，即返回false
    - 将服务端的密钥和客户端时间进行相同的加密算法，得到一个值
    - 把服务端加密后的值和客户端发送过来的加密的密钥进行对比，是否一致
优点：
    - 加密后的密钥每次不同，不易破解
    - 设置有效时长，增加安全性
缺点：
    - 客户端时间和服务端时间不能相差太大，如太大造成认证失败，不易排查问题，最好使用同一时钟服务器进行同步
```
#### 实现
- 客户端

```python
#!/usr/bin/env python
#coding:utf8
 
import json
import platform
import psutil
import urllib,urllib2
import time
import hashlib
from multiprocessing import cpu_count
  
sys_info={}
sys_info['system'] = [] 
sys_info['cpu'] = []
sys_info['mem'] = [] 
sys_info['disk'] = []
sys_info['wip'] = []
sys_info['apikey'] = []
  
#系统信息
system=sys_info['system'].append(platform.uname()[0])
  
#cpu个数
cpu_count=sys_info['cpu'].append(cpu_count())
  
#内存
mem = psutil.virtual_memory()
mem_info = sys_info['mem'].append(mem.total * 1 / (1024**3))
  
#磁盘空间
sdiskusage = psutil.disk_usage('/')
disk_info =sys_info['disk'].append(sdiskusage.total * 1 / (1024**3))
  
#外网ip
def pub_ip():
    url = "http://ip.cip.cc"
    wip = urllib2.urlopen(url).readline().rstrip()
    return  wip
sys_info['wip'].append(pub_ip())
 
#认证密钥
def Api_key():
    client_key = "ddfd-grgf-dsgf-dgfd"    #客户端定义的key
    hash_obj = hashlib.md5()     #使用md5加密，也可以使用sha256
    time_span = time.time()
    hash_obj.update("%s|%f" %(client_key,time_span))    #更新消息，这个update整个文件
    encryption = hash_obj.hexdigest()    #计算消息摘要
    result = "%s|%f" %(encryption,time_span)
    return result
sys_info['apikey'].append(Api_key())
  
if __name__ == '__main__':
    print json.dumps(sys_info,indent=4,ensure_ascii=False)
    data = urllib.urlencode(query=sys_info)
    print data
    respose = urllib.urlopen(url="http://127.0.0.1:8090/serveradd/", data=data)
```
- 服务端

```python
############### urls.py ################
url(r'^serveradd/$','app.views.serveradd'),
 
############### views.py ################
def api_valid(data):
    import time,hashlib
    server_key = "ddfd-grgf-dsgf-dgfd"    #服务端定义的key，也可定义在其他位置
    try:
        encryption , time_span = data.split("|")
        print "------->" ,encryption
        print "------->" ,time_span
        time_span = float(time_span)
        #time.sleep(6)    #模拟认证超时时间
        now_time = time.time()
        if (now_time - time_span) > 5:
            print "超时，认证失败"
            return False
        hash_obj = hashlib.md5()    #使用md5加密
        print "hash_obj------------",hash_obj
         
        hash_obj.update("%s|%f" %(server_key,time_span))     #更新消息，这个update整个文件
        print hash_obj.hexdigest()    #计算消息摘要
        if hash_obj.hexdigest() == encryption:
            return True
        else:
            print "客户端和服务端密钥不一致，认证失败."
            return False
    except Exception,e:
        pass
    return False
     
def api_auth(func):
    def wrapper(req):
        request_dict = req.POST
        api_key = request_dict.get('apikey')
        api_key = api_key[2:-2]
        print api_key
        if not api_key:
            return HttpResponse("Unauthorized.")
        if not api_valid(api_key):
            return HttpResponse("Unauthorized.")
        return func(req)
    return wrapper
 
@api_auth
def serveradd(req):
    if req.method == 'GET':
        print "GET method"
    elif req.method == 'POST':
        print "POST method"
        request_dict = req.POST
        system = request_dict.get('system')
        wip = request_dict.get('wip')
        data = {k: v for k, v in request_dict.items()}
        host = Host.objects.filter(wip=wip).first()
        if not host:
            host = Host(**data)
        host.save(force_insert=True)
    	data = {
            'msg': 'ok',
            'status': 'ok'
            }
    return HttpResponse(data)

```
```
#### 结果说明
1.认证成功，内容如下：
POST method-------
<QueryDict: {u'apikey': [u"['a73421ee7c21d9d590b956f07ed16ca2|1477795149.448697']"], u'mem': [u'[4L]'], u'system': [u"['Darwin']"], u'wip': [u"['14.196.121.237']"], u'disk': [u'[232]'], u'cpu': [u'[4]']}>
----['Darwin']----['14.196.121.237']
 
2.当key不同时，服务端没有收到客户端发送过来的信息，内容如下：
1ab7370c45587a9d4d0d1d8f28bd09d7|1477795289.186698
-------> 1ab7370c45587a9d4d0d1d8f28bd09d7
-------> 1477795289.186698
e9d558f11074410464d1769fb2930e81
客户端和服务端密钥不一致，认证失败。
 
3.当认证时间超时时(这里设置超时时间为5s，模拟发送过来的时间为6s后)，服务端没有收到客户端发送过来的信息，内容如下：
9dc916fbb04c1310cece3aaa5b2d1b7e|1477795521.379039
-------> 9dc916fbb04c1310cece3aaa5b2d1b7e
-------> 1477795521.379039
超时，认证失败
```





