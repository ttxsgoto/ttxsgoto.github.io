---
title: Django Channels
date: 2017-11-26 22:20:43
tags:
  - Channels
categories:
  - Django
---
#### 背景
项目中常使用到实时通信，实时通知等应用，之前使用过第三方提供的接口调用，由于网络原因，不稳定超时，Django 1.8以后支持channel，可实现该功能

#### 定义
Channels基本上就是任务队列：消息被生产者推到通道，然后传递给监听通道的消费者

#### Channels工作层：
1. 接口服务器，Django和用户（浏览器）之间通信的桥梁，包括一个实现WSGI协议的适配器和一个独立的websocket服务器
2. 通道后端， 在接口服务器和worker之间传递消息，由插拔式的python代码和存储组成，存储可以是内存、数据库或者redis，推荐使用redis，兼具其余两者的优点
3. worker，监听所有channel，当有新消息到来时候唤醒功能函数

#### 使用
1. 安装Channels
```python
pip install -U channels
 
# settings.py
 'channels',
```
2. 选择一个通道层(redis)
```python
pip install -U asgi_redis
 
# settings.py
# Channel settings
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "asgi_redis.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get('REDIS_URL', 'redis://redis@127.0.0.1:6379')],
        },
        "ROUTING": "proj.routing.channel_routing",
    },
}
```
3. routing.py
```python
channel_routing = {
    # Wire up websocket channels to our consumers:
    'websocket.connect': consumers.ws_connect,
    'websocket.receive': consumers.ws_receive,
    'websocket.disconnect': consumers.ws_disconnect,
}
 
# - 'websocket.connect': consumers.ws_connect  第一次通过WebSocket连接上时，一条消息被发送到 websocket.connect 通道
# - 'websocket.receive': consumers.ws_receive 每条客户端通过已建立的socket发送的消息都被发送到 websocket.receive通道
# - 'websocket.disconnect': consumers.ws_disconnect 当客户端断开连接时，一条消息被发送到websocket.disconnect通道
```
4. consumers.py
```python
from channels import Group
from channels.sessions import channel_session
import json
 
@channel_session
def ws_connect(message):
    """
    创建message
    """
    group = 'test'
    message.reply_channel.send({"accept": True})
    Group(group, channel_layer=message.channel_layer).add(message.reply_channel)
    message.channel_session['group'] = group    # 通过session保存组信息
 
 
@channel_session
def ws_receive(message):
    """
    发送message
    """
    group = message.channel_session['group']
    message.reply_channel.send({"accept": True})
    data = json.loads(message['text'])
    Group(group, channel_layer=message.channel_layer).send({'text': json.dumps(data.as_dict())})
 
 
@channel_session
def ws_disconnect(message):
    """
    关闭message
    """
    try:
        group = message.channel_session.get('group')
        message.reply_channel.send({"accept": True})
        Group(group, channel_layer=message.channel_layer).discard(message.reply_channel)
    except Exception as e:
        pass

```
5. 运行
```python
import os
import channels.asgi
 
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "proj.settings")
channel_layer = channels.asgi.get_channel_layer()
 
# 测试环境中运行
python manage.py runserver
# 前台接口服务daphne
daphne chat.asgi:channel_layer -p 8888 -b 127.0.0.1 -v2 --access-log=/var/log/asgi.log
# 后台消息消费者
python manage.py runworker
```
6. 部署
```python
# asgi.sh
exec daphne asgi:channel_layer \
    -b 127.0.0.1 \
    -p 8004 -v2 \
    --access-log=/var/logs/asgi.log
# worker.sh
exec python manage.py runworker
# 通过supervisord来启动两个服务
   
# nginx配置支持ws， 这里指定特定url规则来处理ws其他的url不使用ws
location ~ ^/channel/ {
        client_max_body_size 10M;
        proxy_pass         http://127.0.0.1:8004;
        proxy_set_header   Host $host:80;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
 
        # 支持ws配置如下
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```
    7.其他 
    ```python
    # In consumers.py
    from channels import Channel, Group
    from channels.sessions import channel_session
    from channels.auth import channel_session_user, channel_session_user_from_http
     
    # Connected to websocket.connect
    @channel_session_user_from_http	# 在message中添加user对象，也可保存相关的信息到session中
    def ws_add(message):
        # Accept connection
        message.reply_channel.send({"accept": True})
        # Add them to the right group
        Group("chat-%s" % message.user.username[0]).add(message.reply_channel)
     
    # Connected to websocket.receive
    @channel_session_user 
    def ws_message(message):
        Group("chat-%s" % message.user.username[0]).send({
            "text": message['text'],
        })
     
    # Connected to websocket.disconnect
    @channel_session_user
    def ws_disconnect(message):
        Group("chat-%s" % message.user.username[0]).discard(message.reply_channel)
    ```
#### 参考文档
- https://channels.readthedocs.io/en/latest/getting-started.html
- https://blog.heroku.com/in_deep_with_django_channels_the_future_of_real_time_apps_in_django
- https://github.com/jacobian/channels-example
- https://github.com/heshiyou/livelog
- http://masnun.rocks/2016/11/02/deploying-django-channels-using-daphne/




