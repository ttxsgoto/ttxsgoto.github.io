---
title: Python 最简单API实现
date: 2017-03-26 21:51:41
tags:
  - Api
categories:
  - python
---
#### 背景
以前的一个项目中，在做系统资源上报时，因不知道怎么将数据上报给服务端再展示，后来用了一种最直接和不安全的方式，直接写入数据库，这样做很不方便，主要因为需要授权和做相关的防火墙策略（如果有几百上千台client，就算用网段的形式，也需要重复操作），而且如果数据库的信息发生变化时，需要把所有客户端的数据库信息进行修改，比较麻烦；最近学习了一种通过url方式将数据传递给服务端，实现简单的api功能，简单例子如下：

#### 实现
- 客户端

```python
#!/usr/bin/env python
#coding:utf8
 
import json
import platform
import psutil
import urllib,urllib2
from multiprocessing import cpu_count
 
sys_info={}
sys_info['system'] = []
sys_info['cpu'] = []
sys_info['mem'] = []
sys_info['disk'] = []
sys_info['wip'] = []
 
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
 
if __name__ == '__main__':
    print json.dumps(sys_info,indent=4,ensure_ascii=False)
    data = urllib.urlencode(query=sys_info)
    respose = urllib.urlopen(url="http://127.0.0.1:8090/serveradd/", data=data)
```
- 服务端

```python
############### urls.py ################
url(r'^serveradd/$','app.views.serveradd'),
 
############### views.py ################
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
 
############### 结果 ################
POST method
<QueryDict: {u'mem': [u'[4L]'], u'cpu': [u'[4]'], u'disk': [u'[232]'], u'system': [u"['Darwin']"], u'wip': [u"['12.196.9.193']"]}>
----['Darwin']----['12.196.9.193']
```












