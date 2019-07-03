---
title: RabbitMQ
date: 2018-08-25 15:22:15
tags:
  - RabbitMQ
categories:
  - 中间件
---
最近工作需要，需要使用到RabbitMQ，于是理解了一下相关概念和代码测试了一下，记录如下：

#### RabbitMQ简介
AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持Ajax。

#### Queue
RabbitMQ中的消息都只能存储在Queue中，生产者（下图中的P）生产消息并最终投递到Queue中，消费者（下图中的C）可以从Queue中获取消息并消费。
![](https://ttxsgoto.github.io/img/rabbitmq/01.png)
多个消费者可以订阅同一个Queue，这时Queue中的消息会被平均分摊给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。
![](https://ttxsgoto.github.io/img/rabbitmq/02.png)
#### Message acknowledgment(确认机制)
在实际应用中，可能会发生消费者收到Queue中的消息，但没有处理完成就宕机（或出现其他意外）的情况，这种情况下就可能会导致消息丢失。为了避免这种情况发生，我们可以要求消费者在消费完消息后发送一个回执给RabbitMQ，RabbitMQ收到消息回执（Message acknowledgment）后才将该消息从Queue中移除；如果RabbitMQ没有收到回执并检测到消费者的RabbitMQ连接断开，则RabbitMQ会将该消息发送给其他消费者（如果存在多个消费者）进行处理。这里不存在timeout概念，一个消费者处理消息时间再长也不会导致该消息被发送给其他消费者，除非它的RabbitMQ连接断开。
这里会产生另外一个问题，如果我们的开发人员在处理完业务逻辑后，忘记发送回执给RabbitMQ，这将会导致严重的bug——Queue中堆积的消息会越来越多；消费者重启后会重复消费这些消息并重复执行业务逻辑,另外pub message是没有ack的。
rabbitmq_producer_ack.py
```python
#!/usr/bin/env python
# coding: utf-8
 
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建
channel.queue_declare(queue="ttxsgoto")
 
# 设置指定队列名称，body往队列中发送消息
for i in range(20):
    channel.basic_publish(exchange='',
                          routing_key="ttxsgoto",
                          body="Hello World!--{}".format(i))
    print("Sent 'Hello World!' - {} ".format(i))
 
connection.close()
```
rabbitmq_consumer_ack.py
```python
#!/usr/bin/env python
# coding: utf-8
 
import time
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建
channel.queue_declare(queue="ttxsgoto")
 
 
def callback(ch, method, properties, body):
    print("Received %s" % body)
    time.sleep(1)
    print("ok")
    ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue="ttxsgoto",
                      no_ack=False)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
#### Message durability(持久化)
如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，我们可以将Queue与Message都设置为可持久化的（durable），这样可以保证绝大部分情况下我们的RabbitMQ消息不会丢失。
rabbitmq_producer_ack_durable.py
```python
#!/usr/bin/env python
# coding: utf-8
# 消息队列是可以做持久化，如果我们在生产消息的时候就指定某条消息需要做持久化，那么RabbitMQ发现有问题时，就会将消息保存到硬盘，持久化下来
# 此时rabbitmq down掉时，再启动 队列和数据也都是存在的，如果不持久化，down掉后队列就没有了
 
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建,指定队列持久化
channel.queue_declare(queue="ttxsgoto01", durable=True)
 
# 设置指定队列名称，body往队列中发送消息
for i in range(20):
    channel.basic_publish(exchange='',
                          routing_key="ttxsgoto01",
                          body="Hello World!--{}".format(i),
                          properties=pika.BasicProperties(delivery_mode=2, ))  # 指定消息持久化
    print("Sent 'Hello World!' - {} ".format(i))
 
# 关闭连接
connection.close()
 
```
rabbitmq_consumer_ack_durable.py
```python
#!/usr/bin/env python
# coding: utf-8
# 消息队列是可以做持久化，如果我们在生产消息的时候就指定某条消息需要做持久化，那么RabbitMQ发现有问题时，就会将消息保存到硬盘，持久化下来
 
import time
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建,队列持久化
channel.queue_declare(queue="ttxsgoto01", durable=True)
 
 
def callback(ch, method, properties, body):
    print("Received %s" % body)
    time.sleep(1)
    print("ok")
    ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue="ttxsgoto01",
                      no_ack=False)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
说明：消息队列是可以做持久化，如果我们在生产消息的时候就指定某条消息需要做持久化，那么RabbitMQ发现有问题时，就会将消息保存到硬盘，持久化下来；此时可以通过rabbitmq web管理页面看到队列ttxsgoto01的Parameters有一个D属性，表示durable=True

#### Prefetch count(每发送数)
如果有多个消费者同时订阅同一个Queue中的消息，Queue中的消息会被平摊给多个消费者。这时如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置prefetchCount来限制Queue每次发送给每个消费者的消息数，比如我们设置prefetchCount=1，则Queue每次给每个消费者发送一条消息；消费者处理完这条消息后Queue会再给该消费者发送一条消息
rabbitmq_producer_ack_durable_qos.py
```python
#!/usr/bin/env python
# coding: utf-8
# 消息队列是可以做持久化，如果我们在生产消息的时候就指定某条消息需要做持久化，那么RabbitMQ发现有问题时，就会将消息保存到硬盘，持久化下来
# 此时rabbitmq down掉时，再启动 队列和数据也都是存在的，如果不持久化，down掉后队列就没有了
 
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建,指定队列持久化
channel.queue_declare(queue="ttxsgoto02", durable=True)
 
# 设置指定队列名称，body往队列中发送消息
for i in range(20):
    channel.basic_publish(exchange='',
                          routing_key="ttxsgoto02",
                          body="Hello World!--{}".format(i),
                          properties=pika.BasicProperties(delivery_mode=2, ))  # 指定消息持久化
    print("Sent 'Hello World!' - {} ".format(i))
 
# 关闭连接
connection.close()
 
```
rabbitmq_consumer_ack_durable_qos.py
```python
#!/usr/bin/env python
# coding: utf-8
 
import time
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 通过频道创建队列，如果有则忽略，没有则创建,队列持久化
channel.queue_declare(queue="ttxsgoto02", durable=True)
 
 
def callback(ch, method, properties, body):
    print("Received %s" % body)
    time.sleep(1)
    print("ok")
    ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 表示谁来获取，不再按照奇偶数排列
channel.basic_qos(prefetch_count=1)
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue="ttxsgoto02",
                      no_ack=False)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
#### Exchange(交换器)
在上一节我们看到生产者将消息投递到Queue中，实际上这在RabbitMQ中这种事情永远都不会发生。实际的情况是，生产者将消息发送到Exchange（交换器，下图中的X），由Exchange将消息路由到一个或多个Queue中（或者丢弃）。
![](https://ttxsgoto.github.io/img/rabbitmq/03.png)
#### routing key
生产者在将消息发送给Exchange的时候，一般会指定一个routing key，来指定这个消息的路由规则，而这个routing key需要与Exchange Type及binding key联合使用才能最终生效。（routing key + exchange type + binding key）
在Exchange Type与binding key固定的情况下（在正常使用时一般这些内容都是固定配置好的），我们的生产者就可以在发送消息给Exchange时，通过指定routing key来决定消息流向哪里。
RabbitMQ为routing key设定的长度限制为255 bytes
#### Binding(绑定到Queue)
RabbitMQ中通过Binding将Exchange与Queue关联起来，这样RabbitMQ就知道如何正确地将消息路由到指定的Queue了。
![](https://ttxsgoto.github.io/img/rabbitmq/04.png)
#### Binding key
在绑定（Binding）Exchange与Queue的同时，一般会指定一个binding key；消费者将消息发送给Exchange时，一般会指定一个routing key；当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。
在绑定多个Queue到同一个Exchange的时候，这些Binding允许使用相同的binding key。
binding key 并不是在所有情况下都生效，它依赖于Exchange Type，比如fanout类型的Exchange就会无视binding key，而是将消息路由到所有绑定到该Exchange的Queue。

#### Exchange Types
RabbitMQ常用的Exchange Type有fanout、direct、topic、headers这四种,一般headers使用较少，不做演示说明，下面分别进行介绍
##### fanout
fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中。
![](https://ttxsgoto.github.io/img/rabbitmq/05.png)
上图中，生产者（P）发送到Exchange（X）的所有消息都会路由到图中的两个Queue，并最终被两个消费者（C1与C2）消费。
rabbitmq_producer_fanout.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = fanout ：任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上
 
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host='127.0.0.1', port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 指定exchange和对应的类型
channel.exchange_declare(exchange="test_fanout",
                         exchange_type='fanout')
 
# 设置exchange，没有指定routing_key，队列随机
for i in range(20):
    channel.basic_publish(exchange='test_fanout',
                          routing_key="abc",
                          body="Hello World!--{}".format(i),
                          properties=pika.BasicProperties(delivery_mode=2, ))  # 指定消息持久化
    print("Sent 'Hello World!' - {} ".format(i))
 
# 关闭连接
connection.close()
 
```
rabbitmq_consumer_fanout.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = fanout ：任何发送到Fanout Exchange的消息都会被转发到与该Exchange绑定(Binding)的所有Queue上
 
import pika
import time
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
channel.exchange_declare(exchange="test_fanout",  # 创建一个exchange
                         exchange_type="fanout")  # 任何发送到fanout exchange的消息都会被转发到和exchange绑定的queue上
 
# 随机创建队列
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue
 
# 绑定，exchange绑定后端队列
channel.queue_bind(exchange="test_fanout",
                   queue=queue_name)
 
 
def callback(ch, method, properties, body):
    print("Received %s" % body)
    time.sleep(1)
    # ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
##### direct
direct类型的Exchange路由规则也很简单，它会把消息路由到那些binding key与routing key完全匹配的Queue中。
![](https://ttxsgoto.github.io/img/rabbitmq/06.png)
以上图的配置为例，我们以routingKey=”error”发送消息到Exchange，则消息会路由到Queue1（amqp.gen-S9b…，这是由RabbitMQ自动生成的Queue名称）和Queue2（amqp.gen-Agl…）；如果我们以routingKey=”info”或routingKey=”warning”来发送消息，则消息只会路由到Queue2。如果我们以其他routingKey发送消息，则消息不会路由到这两个Queue中。
rabbitmq_producer_direct.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = direct：任何发送到Direct Exchange的消息都会被转发到RouteKey中指定的Queue上(关键字发送)
# 队列绑定关键字，发送者将数据关键字发送到消息Exchange，Exchange根据关键字判定应该将数据发送至指定队列
# 结论：当我们将发布者的key设置成Error的时候两个队列对可以收到Exchange的消息，当我们将key设置成info后，只有订阅者1可以收到Exchange的消息
 
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 指定exchange和对应的类型
channel.exchange_declare(exchange="test_direct",
                         exchange_type='direct')
typeinfo = 'info'
 
# 设置exchange，没有指定routing_key，队列指定关键字
for i in range(20):
    channel.basic_publish(exchange='test_direct',
                          routing_key=typeinfo,
                          body="Hello World!--{}".format(i),
                          properties=pika.BasicProperties(delivery_mode=2, ))  # 指定消息持久化
    print("Sent 'Hello World!' - {} ".format(i))
 
# 关闭连接
connection.close()
 
```
rabbitmq_consumer_direct1.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = direct：任何发送到Direct Exchange的消息都会被转发到RouteKey中指定的Queue上(关键字发送)
# 队列绑定关键字，发送者将数据关键字发送到消息Exchange，Exchange根据关键字判定应该将数据发送至指定队列
 
import pika
import time
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
channel.exchange_declare(exchange="test_direct",  # 创建一个exchange
                         exchange_type="direct")
# 随机创建队列
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue
 
typeinfo = ['error', 'info', ]
 
# 绑定，exchange绑定后端队列
for type1 in typeinfo:
    channel.queue_bind(exchange="test_direct", queue=queue_name, routing_key=type1)
 
 
def callback(ch, method, properties, body):
    print("Received %s --- %s" % (method.routing_key, body))
    time.sleep(1)
    # ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
rabbitmq_consumer_direct2.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = direct：任何发送到Direct Exchange的消息都会被转发到RouteKey中指定的Queue上(关键字发送)
# 队列绑定关键字，发送者将数据关键字发送到消息Exchange，Exchange根据关键字判定应该将数据发送至指定队列
 
import pika
import time
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
channel.exchange_declare(exchange="test_direct",  # 创建一个exchange
                         exchange_type="direct")
# 随机创建队列
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue
 
typeinfo = ['error', ]
 
# 绑定，exchange绑定后端队列
for type1 in typeinfo:
    channel.queue_bind(exchange="test_direct", queue=queue_name, routing_key=type1)
 
 
def callback(ch, method, properties, body):
    print("Received %s --- %s" % (method.routing_key, body))
    time.sleep(1)
    # ch.basic_ack(delivery_tag=method.delivery_tag)  # 向生产者发送消费完毕的确认消息，然后生产者将该条消息从队列中剔除
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
结论：当我们将发布者的key设置成error的时候两个队列对可以收到Exchange的消息，当我们将key设置成info后，只有订阅者1可以收到Exchange的消息。
##### topic
direct类型的Exchange路由规则是完全匹配binding key与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但这里的匹配规则有些不同，它约定：
- routing key为一个句点号“. ”分隔的字符串（我们将被句点号“. ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”

- binding key与routing key一样也是句点号“. ”分隔的字符串

- binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）

![](https://ttxsgoto.github.io/img/rabbitmq/07.png)

以上图中的配置为例，routingKey=”quick.orange.rabbit”的消息会同时路由到Q1与Q2，routingKey=”lazy.orange.fox”的消息会路由到Q1，routingKey=”lazy.brown.fox”的消息会路由到Q2，routingKey=”lazy.pink.rabbit”的消息会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）；routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何bindingKey。
rabbitmq_producer_topic.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = topic：任何发送到Topic Exchange的消息都会被转发到所有关心RouteKey中指定话题的Queue上(模糊匹配)
# 在topic类型下，可以让队列绑定几个模糊的关键字，之后发送者将数据发送到exchange，exchange将传入"路由值"和"关键字"进行匹配，匹配成功，则将数据发送到指定队列
# # ：表示可以匹配0个或多个单词
# * ：表示只能匹配一个单词
import sys
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 指定exchange和对应的类型
channel.exchange_declare(exchange="test_topic",
                         exchange_type='topic')
 
routing_key = sys.argv[1] if len(sys.argv) > 1 else 'anonymous'
message = ''.join(sys.argv[2:]) or 'Hello Chengdu!'
 
# 设置exchange，没有指定routing_key，队列指定关键字
channel.basic_publish(exchange='test_topic', routing_key=routing_key, body=message)
 
print(" Sent routing_key:%s ——> body:%s " % (routing_key, message))
# 关闭连接
connection.close()
 
```
rabbitmq_consumer_topic.py
```python
#!/usr/bin/env python
# coding: utf-8
# exchange type = topic：任何发送到Topic Exchange的消息都会被转发到所有关心RouteKey中指定话题的Queue上(模糊匹配)
# 在topic类型下，可以让队列绑定几个模糊的关键字，之后发送者将数据发送到exchange，exchange将传入"路由值"和"关键字"进行匹配，匹配成功，则将数据发送到指定队列
 
import sys
import pika
 
connection = pika.BlockingConnection(pika.ConnectionParameters(host="127.0.0.1", port=5672))
 
# 创建一个频道
channel = connection.channel()
 
# 创建一个exchange,并指定类型
channel.exchange_declare(exchange="test_topic",
                         exchange_type="topic")
# 随机创建队列
result = channel.queue_declare(exclusive=True)
 
queue_name = result.method.queue
binding_keys = sys.argv[1:]
 
if not binding_keys:
    sys.stderr.write("Usage: %s [binding_key]...\n" % sys.argv[0])
    sys.exit(0)
 
for binding_key in binding_keys:
    channel.queue_bind(exchange="test_topic", queue=queue_name, routing_key=binding_key)
 
 
def callback(ch, method, properties, body):
    print("Received %s -----%s " % (method.routing_key, body))
 
 
# 从队列中取消息
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)  # 如果no_ack=False,当消费者down掉，rabbitmq会重新将该任务添加到队列中
 
print("Waiting for messages,To exit press CTRL + C")
channel.start_consuming()
 
```
#### 常用命令
```
添加用户：
rabbitmqctl add_user abc abc
 
添加权限：
rabbitmqctl set_permissions -p "/" abc ".*" ".*" ".*"
 
设置用户标签：
rabbitmqctl set_user_tags abc administrator
 
删除用户：
rabbitmqctl delete_user guest
 
修改密码：
rabbitmqctl change_password   username  newpassword
 
list_users
add_vhost   vhostpath
rabbitmqctl list_user_permissions abc  
list_queues 
list_exchanges
list_bindings
```