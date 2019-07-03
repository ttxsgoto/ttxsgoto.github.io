---
title: 运维 Linux(fg_jobs)
date: 2017-05-15 22:31:08
tags:
  - fg
categories:
  - 运维
---
#### 背景
在启动程序后，往往需要查看启动日志是否正常，有无报错，而有时日志量很大（刷屏），不易看清楚日志，这时就需要停下来，之前一直使用CRTL+C 直接退出，而后想看接下来的日志的话，日志已经刷好远了。。。。，今天发现一个将程序存放到后台的功能，需要时再调回前台查看就可以了，刚开始学linux时 知道有这功能，工作后一直没有用；今天用起来了，记录如下：
ping www.baidu.com  然后按 CRTL+Z,如下放到后台运行 
```python
root@hadoop1:~# ping www.baidu.com
PING www.a.shifen.com (180.97.33.107) 56(84) bytes of data.
64 bytes from 180.97.33.107: icmp_seq=1 ttl=54 time=42.5 ms
64 bytes from 180.97.33.107: icmp_seq=2 ttl=54 time=42.4 ms
^Z
[1]+  Stopped                 ping www.baidu.com
```
ping blog.51cto.com 
```python
root@hadoop1:~# ping ttxsgoto.github.io
PING sni.github.map.fastly.net (151.101.73.147): 56 data bytes
64 bytes from 151.101.73.147: icmp_seq=0 ttl=54 time=154.436 ms
64 bytes from 151.101.73.147: icmp_seq=1 ttl=54 time=145.728 ms
64 bytes from 151.101.73.147: icmp_seq=2 ttl=54 time=145.869 ms
^Z
[2]+  Stopped                 ping ttxsgoto.github.io
```
ping docs.python.org
```python
root@hadoop1:~# ping docs.python.org
PING prod.python.map.fastlylb.net (151.101.16.223) 56(84) bytes of data.
64 bytes from 151.101.16.223: icmp_seq=1 ttl=44 time=513 ms
64 bytes from 151.101.16.223: icmp_seq=2 ttl=44 time=513 ms
^Z
[3]+  Stopped                 ping docs.python.org
```
jobs 查看目前运行的后台程序
```python
root@hadoop1:~# jobs 
[1]   Stopped                 ping www.baidu.com
[2]-  Stopped                 ping ttxsgoto.github.io
[3]+  Stopped                 ping docs.python.org
```
fg 将最近后台运行程序(+)前台执行，这里为3 —ping docs.python.org
```python
root@hadoop1:~# fg 
ping docs.python.org
64 bytes from 151.101.16.223: icmp_seq=4 ttl=44 time=491 ms
64 bytes from 151.101.16.223: icmp_seq=5 ttl=44 time=488 ms
64 bytes from 151.101.16.223: icmp_seq=7 ttl=44 time=489 ms
64 bytes from 151.101.16.223: icmp_seq=8 ttl=44 time=496 ms
```
可以看到基本上是从放入后台这个时间点运行到前台的
fg 1  将系列号为1的后台程序放入前台执行
```python
root@hadoop1:~# fg 1
ping www.baidu.com
64 bytes from 180.97.33.107: icmp_seq=12 ttl=54 time=41.1 ms
64 bytes from 180.97.33.107: icmp_seq=13 ttl=54 time=42.1 ms
64 bytes from 180.97.33.107: icmp_seq=14 ttl=54 time=42.8 ms
64 bytes from 180.97.33.107: icmp_seq=15 ttl=54 time=41.2 ms
```
功能简单，但挺实用，记录一下！
