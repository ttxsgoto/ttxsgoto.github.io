---
title: Python Threading
date: 2017-04-17 20:37:11
tags:
  - Threading
categories:
  - python
---
#### 线程5种状态
- 新建
- 就绪
- 运行
- 阻塞
- 死亡

阻塞的三种情况：
- 同步阻塞：是指处于竞争锁定的状态，线程请求锁定时将进入这个状态，一旦成功获得锁定又恢复到运行状态；
- 等待阻塞：是指等待其他线程通知的状态，线程获得条件锁定后，调用“等待”将进入这个状态，一旦其他线程发出通知，线程将进入同步阻塞状态，再次竞争条件锁定；
- 其他阻塞：是指调用time.sleep()、anotherthread.join()或等待IO时的阻塞，这个状态下线程不会释放已获得的锁定。

#### GIL
在python虚拟机中的访问由全局解释器锁（GIL）控制，由于有了这把锁能保证同一时刻只有一个线程在运行，在多线程环境下，python虚拟机按照如下方式运行：
1. 设置GIL
2. 切换到一个线程中运行
3. 运行：
    a指定数量的字节码的指令，或者
    b线程主动让出控制（可以调用time.sleep(0)）
4. 把线程设置为睡眠状态
5. 解锁GIL
6. 在重复以上所有步骤

#### threading.Thread模块
threading.Thread 作用：创建线程实例
```python
- start()    开始一个线程的执行
- run()    定义线程的功能的函数（一般会被子类重写）
- join(timeout=None)  程序挂起，直到子线程结束，在执行主线程，如果给了timeout，则最多阻塞timeout秒
- getName()    返回线程的名字
- setName()    设置线程的名字
- isAlive()    检查线程是否成活
- isDaemon()   是否等待线程执行完成后在执行主进程，默认为false
- setDeaemon() 设置为daemoe模式，setDeaemon(True)，不等子线程执行完成，直接执行主线程；如果在主线程没有结束时，子线程会执行直到主线程结束，子线程也结束
```
#### 多线程运行方式
##### 函数调用
target传入函数名称，args传递给函数的参数
```python
#!/usr/bin/env python
# encoding: utf-8
from threading import Thread
from time import sleep,ctime,sleep
 
def myfunction(arg):
    for item in range(10):
        print item
        sleep(1)
print "before"
 
t1 = Thread(target=myfunction,args=('hello',))
print t1.getName() #Thread-1
print t1.isDaemon() #默认为false
#t1.setDaemon(True)  #设置为setDaemon模式
t1.start()
t1.join(timeout=5)
print "after"
print "Over"
sleep(5)
```
##### 类继承调用
使用Threading创建线程，从threading.Thread继承，然后重写init方法和run方法
```python
#!/usr/bin/env python
#coding:utf-8
 
import time
from threading import Thread
 
class mythread(Thread): #继承父类的threading.Thread
    def __init__(self,threadid,threadname,counter):
        Thread.__init__(self)
        self.threadid = threadid
        self.threadname = threadname
        self.counter = counter
         
    def run(self): #把要执行的代码写在run函数中，线程创建后会直接运行run函数
        print "Starting %s \n" %self.threadname
        print_time(self.name,self.counter,self.threadid) #函数调用
        print "Exiting %s" %self.threadname
         
def print_time(threadName,delay,counter):
    while counter :
        time.sleep(delay)
        print counter,"%s: %s" %(threadName,time.ctime())
        counter -=1
         
t1 = mythread(2,"线程一",1)
t1.start()
```
#### 生产者消费者模型
优点：
1. 解耦：两者都只依赖于缓冲区，不相互依赖
2. 支持并发：生产者把制造出来的数据往缓冲区一丢，就可以再去生产下一个数据，即不用依赖消费者的处理速度
3. 支持忙闲不均

```python
#!/usr/bin/env python
#coding:utf-8
import threading
import time,random
import Queue
 
def proudcer(name,que):
    while True:
        if que.qsize() < 3:
            que.put('生产包子')
            print "%s:生产了一个包子...." %name
        time.sleep(random.randrange(5))
         
def consumer(name,que):
    while True:
        try:
            que.get_nowait() #不等待队列是否有值
            print "%s:消费了一个包子...." %name
        except Exception:
            print u'没有包子可消费了 .....'
        time.sleep(random.randrange(3))
         
#实例化生产者
q = Queue.Queue()
p1 = threading.Thread(target=proudcer,args=("生产者01",q))
p2 = threading.Thread(target=proudcer,args=("生产者02",q))
p1.start()
p2.start()
 
#实例化消费者
c1 = threading.Thread(target=consumer,args=("消费者01",q))
c2 = threading.Thread(target=consumer,args=("消费者02",q))
c1.start()
c2.start()
```
#### 线程安全
线程锁,保证数据安全，多个线程都同时修改某个变量，有可能出现问题，Lock处于锁定状态时，不被特定的线程拥有。Lock包含两种状态——锁定和非锁定，以及两个基本的方法

运行：任何时刻只能有一个线程在执行

定义和方法:

        lock = threading.Lock() #线程锁定义
        lock.acquire() #获取锁，开始独占cpu
        slock.release() #释放锁，可以被其他使用cpu资源,必须在获得锁定后再使用，否则抛出异常
        slock.locked()  #主程序判断locked()状态
        Lock= threading.RLock() #可重入锁，多把锁时引用，释放时也应该释放对应的多把锁
        lock = threading.BoundedSemaphore(4) #同时允许多少个线程进行数据修改信号量：同一时刻，允许几个线程运行
```python
#!/usr/bin/env python
#coding:utf-8
import threading
import time
 
data = 0
lock = threading.Lock() #定义锁
 
def func():
    global data
    print "%s acquire lock ..." %threading.currentThread().getName()
    #print "acquire-----%s" %lock.acquire()
    if lock.acquire(): #调用锁，返回是否获得锁
        print "%s get the lock." %threading.currentThread().getName()
        data += 1
        print data
        time.sleep(2)
        print "%s release lock..." %threading.currentThread().getName()
        lock.release() # 调用release()将释放锁
         
t1 = threading.Thread(target=func)
t2 = threading.Thread(target=func)
t3 = threading.Thread(target=func)
t1.start()
t2.start()
t3.start()
```
#### Event
```
event = threading.Event()
event.wait()    #将阻塞线程放置为阻塞状态，等待设置处理
event.set()   #event内置了一个初始为false的标示，当调用set时，设置为true，开始处理
event.clear()    #将set标示位清空，设置为false
event.isSet()    #判断set标示位是否为true
```
#### 参考链接
http://www.cnblogs.com/huxi/archive/2010/06/26/1765808.html

