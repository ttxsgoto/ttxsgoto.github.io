---
title: asyncio 基本用法
date: 2018-08-10 11:58:11
tags:
  - asyncio
categories:
  - python
---
### async/await
#### 协程
同时开启多个任务，但一次只执行一个，当执行的任务遇到阻塞，就切换到下一个任务继续执行，
以期节省阻塞所占用的时间

要实现异步处理，需要有挂起的操作，当一个任务需要等待 IO 结果的时候，可以挂起当前任务，转而去执行其他任务

1. event_loop： 事件循环，把一些函数注册到这个事件循环中，当条件满足时，就执行对应的方法
2. coroutine： 协程，可以看做一个协程对象类型，把协程对象注册到事件循环中，满足条件时被调用； async关键字来定义一个方法，这个方法在调用时不会立刻执行，而是返回一个协程对象
3. task： 任务，它是对协程对象的进一步封装，包含了任务的各个状态
4. future： 代表即将执行或者还没有执行的任务的结果，可以等同与task
5. async： 用于定义协程， 协程函数不能直接调用，直接调用协程函数得到的是协程对象(<coroutine object execute at 0x10359c938>)
6. await：用来将阻塞方法进行挂起

#### await 后的对象必须为如下的格式
- 原生的coroutine对象
- 一个由types.coroutine()修饰的生成器，这个生成器可以返回 coroutine 对象
- 一个包含 __await 方法的对象返回的一个迭代器。


#### asyncio模块实例
##### 基础用法
```python
import asyncio
 
async def execute(x):
    print('No. ', x)
 
coroutine = execute(2)
 
print('Coroutine-->', coroutine)    # 协程对象
print('After calling execute')
 
# 创建事件循环
loop = asyncio.get_event_loop()
 
# 方法一:将coroutine封装成task对象
# task = loop.create_task(coroutine)
# 方法二:通过 asyncio 的 ensure_future() 方法, 将coroutine封装成task对象,这样可以不借助于loop来定义,
#即使没有声明loop也可以提前定义好task对象
task = asyncio.ensure_future(coroutine)
task.add_done_callback(callback)    # task添加回调函数(通过task.result()获取返回值)
 
print('Task:', task)  # 查看任务task的状态
# Task: <Task pending coro=<execute() ...
# 将协程对象注册到事件循环中
loop.run_until_complete(task)
 
print('Task:', task)
# Task: <Task finished coro=<execute() done ...
print('Task result: ', task.result()) # task在fiinished状态时,直接读取task的result方法，得到返回值
# loop.run_until_complete(coroutine)
print('After calling loop')

```
##### 多任务执行
定义一个task列表，然后使用asyncio的wait()方法即可执行
```python
import asyncio
import requests
 
now = time.time()
 
async def num(number):
    await asyncio.sleep(number)
    return number
 
tasks = [asyncio.ensure_future(num(i)) for i in range(5)]
print('Tasks-', tasks)
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))    # asyncio.wait()用于执行tasks
print('Tasks-', tasks)
for task in tasks:
    print('Task Result: ', task.result())
 
print('time-->', time.time() - now)
```
##### 协程嵌套
即一个协程中await了另外一个协程
```python
import asyncio
import time
 
now = time.time()
 
async def num(number):
    await asyncio.sleep(number)
    return number
 
async def main():
    tasks = [asyncio.ensure_future(num(i)) for i in range(4)]

    # _tasks = await asyncio.gather(*tasks) #  asyncio.gather创建协程对象,await的返回值就是协程运行的结果
    _tasks, pending = await asyncio.wait(tasks) # asyncio.wait挂起协程,Returns two sets of Future: (done, pending).
    print('-----', _tasks )

    for task in _tasks: # 在函数里面返回结果
        print(('wait result task->', task.result()))
        print('gather result task->', task)
    return _tasks
 
loop = asyncio.get_event_loop()
results = loop.run_until_complete(main())
 
for result in results:
    print(('wait result task->', result.result()))
    # print('gather result task->', result)
 
print('TIME:', time.time() - now)
```












