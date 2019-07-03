---
title: 运维 Redis进阶
date: 2018-09-28 21:14:53
tags:
  - Redis
categories:
  - 运维
---
Redis的数据存在内存中， 读写速度非常快，常用于缓存数据，来提高系统的高性能和高并发

#### 使用redis目的
- 高性能：将数据缓存在redis中，访问数据从缓存中取，不直接访问数据库，提高页面响应效率
- 高并发：在大的并发情况下，直接操作缓存能够承受的请求远大于直接访问数据库，这时我们需要使用redis做一个缓冲操作，让请求先访问到redis，而不是直接访问数据库

#### 一般应用场景

- 缓存-热数据：需要执行耗时久，计算结果不频繁变动的sql查询
- 异步队列
- 计数器：如统计点击数，INCRBY
- 分布式锁与单线程机制
- 最新列表：使用LPUSH命令构建List
- 排行榜应用： 使用ZADD(有续集，sorted set)
- 位操作(大数据处理)

#### 单线程的redis为什么快
- 纯内存操作
- 单线程操作，避免了频繁的上下文切换
- 采用非阻塞I/O多路复用机制
    
    ```
    在redis服务端，启用了I/O多路复用机制，将其置于队列中，然后文件事件分发器依次去队列中去取，转发到不同的事件处理器中处理
    ```

#### 常见数据结构和使用场景
- String 

常用的命令： set、get、decr、incr、mget、mset
String 数据结构是简单的 Key-Value 类型，Value 可为字符和数值和其他类型的值
```python
# 设置和获取key-value
set mykey ttxsgoto
get mykey
 
# 原子递增
set counter 100
incr counter ((integer) 101)
 
# 一次性存储和获取多个key对应的值,mget 命令返回由值组成的数组
127.0.0.1:6379> mset a 10 b 20 c 30
127.0.0.1:6379> mget a b c
1) "10"
2) "20"
3) "30"
127.0.0.1:6379> keys *
1) "ttxs"
2) "c"
3) "a"
4) "counter"
5) "b"
6) "mykey"
127.0.0.1:6379> exists a
(integer) 1
127.0.0.1:6379> exists x
(integer) 0
127.0.0.1:6379> type a
string
127.0.0.1:6379> del a
(integer) 1
 
# 设置过期时间
127.0.0.1:6379> get b
"20"
127.0.0.1:6379> expire b 5
(integer) 1
127.0.0.1:6379> get b
(nil)
127.0.0.1:6379> set bbb 1000 ex 100
OK
# 查看过期时间
127.0.0.1:6379> ttl bbb
(integer) 95
```

- Hash

常用命令：hget、hset、 hmget、hmset、hgetall
Hash 是一个 String 类型的 Field 和 Value 的映射表，Hash 特别适合用于存储对象；后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值
```python
# 由键值对组成
127.0.0.1:6379> hmset user:1000 username antirez birthyear 1977 verified 1
OK
127.0.0.1:6379> hget user:1000 username
"antirez"
127.0.0.1:6379> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
 
# 返回多个值
127.0.0.1:6379> hmget user:1000 username birthyear xxx
1) "antirez"
2) "1977"
3) (nil)
 
```
- List

常用命令：lpush、rpush、lpop、rpop、lrange
List 就是链表，Redis List 的应用场景非常多，也是 Redis 最重要的数据结构之一
Redis List 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销
另外可以通过 lrange 命令，就是从某个元素开始读取多少个元素，可以基于 List 实现分页查询

```python
# lpush 向list的左边（头部）添加一个新元素
# rpush 向list的右边（尾部）添加一个新元素
# lrange 从list中取出一定范围的元素
127.0.0.1:6379> rpush mylist A
(integer) 1
127.0.0.1:6379> rpush mylist B
(integer) 2
127.0.0.1:6379> lpush mylist first
(integer) 3
# 
127.0.0.1:6379> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
# 写入多个值
127.0.0.1:6379> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
127.0.0.1:6379> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
 
127.0.0.1:6379> rpush mylist001 a b c
# 删除元素并同时返回删除的值
127.0.0.1:6379> rpop mylist001
"c"
 
# ltrim把list从左边截取指定长度
127.0.0.1:6379> ltrim mylist 0 2
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "first"
2) "A"
3) "B"
# 查看list中元素的个数
127.0.0.1:6379> llen mylist
(integer) 6
127.0.0.1:6379> del mylist
(integer) 1
```
- Set

常用命令：sadd、spop、smembers、sunion
Set 对外提供的功能与 List 类似是一个列表的功能，特殊之处在于 Set 是可以自动排重
当你需要存储一个列表数据，又不希望出现重复数据时，可以使用Set，同时也支持交集、并集、差集操作
```python
# 添加
127.0.0.1:6379> sadd myset 1 2 3
(integer) 3
# 查看所有
127.0.0.1:6379> smembers myset
1) "1"
2) "2"
3) "3"
# 检查元素是否存在
127.0.0.1:6379> sismember myset 1
(integer) 1
127.0.0.1:6379> sismember myset 4
(integer) 0
 
```
- Sorted Set

常用命令：zadd、zrange、zrem、zcard 
和 Set 相比，Sorted Set 增加了一个权重参数 Score，使得集合中的元素能够按 Score 进行有序排列
```python
# zadd 添加
127.0.0.1:6379> zadd hackers 1940 "Alan Kay"
(integer) 1
127.0.0.1:6379> zadd hackers 1957 "Sophie Wilson"
(integer) 1
127.0.0.1:6379> zadd hackers 1953 "Richard Stallman"
(integer) 1
127.0.0.1:6379> zadd hackers 1949 "Anita Borg"
(integer) 1
127.0.0.1:6379> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
127.0.0.1:6379> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
127.0.0.1:6379> zadd hackers 1916 "Claude Shannon"
(integer) 1
127.0.0.1:6379> zadd hackers 1969 "Linus Torvalds"
(integer) 1
127.0.0.1:6379> zadd hackers 1912 "Alan Turing"
(integer) 1
 
# 查看数据
127.0.0.1:6379> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
 
# 反向查看数据
127.0.0.1:6379> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
 
# 查看对应的得分
127.0.0.1:6379> zrange hackers 0 -1 withscores
 1) "Alan Turing"
 2) "1912"
 3) "Hedy Lamarr"
 4) "1914"
 5) "Claude Shannon"
 6) "1916"
 7) "Alan Kay"
 8) "1940"
 9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
  
# 小于1950的数据
127.0.0.1:6379> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```
#### 过期策略以及内存淘汰机制
redis采用的是定期删除+惰性删除策略
- 定期删除：Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 Key，检查其是否过期，如果过期就删除。注意这里是随机抽取的。为什么要随机？假如 Redis 存了几十万个 Key ，每隔 100ms 就遍历所有的设置过期时间的 Key 的话，就会给 CPU 带来很大的负载
- 惰性删除 ：定期删除可能会导致很多过期 Key 到了时间并没有被删除掉。所以就有了惰性删除，也就是说在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除

- 内存淘汰机制： 在redis配置文件中配置 # maxmemory-policy volatile-lru
Redis 提供 6 种数据淘汰策略：


    - volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
    - volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
    - volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
    - allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）
    - allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
    - no-enviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错

#### 持久化机制
Redis 的一种持久化方式叫快照（snapshotting，RDB），另一种方式是只追加文件（append-only file，AOF）。

- RDB 快照持久化是 Redis 默认采用的持久化方式，在 redis.conf 配置文件中默认有此下配置：
```python
save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
```
- AOF
与快照持久化相比，AOF 持久化的实时性更好，因此已成为主流的持久化方案
默认情况下 Redis 没有开启 AOF（append only file）方式的持久化，可以通过 appendonly 参数开启：
```python
appendonly yes
```
开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件。

AOF 文件的保存位置和 RDB 文件的位置相同，都是通过 dir 参数设置的，默认的文件名是 appendonly.aof。

在 Redis 的配置文件中存在三种不同的 AOF 持久化方式，它们分别是：
```python
appendfsync always     #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
appendfsync no      #让操作系统决定何时进行同步
```
为了兼顾数据和写入性能，用户可以考虑 appendfsync everysec 选项 ，让 Redis 每秒同步一次 AOF 文件，Redis 性能几乎没受到任何影响。

- Redis 4.0 对于持久化机制的优化

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 aof-use-rdb-preamble 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。

这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。

当然缺点也是有的，AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差。

#### 缓存雪崩
当缓存失效(过期)后引起系统性能急剧下降的情况

解决方案：
- 更新锁机制
    ```
    对缓存更新操作进行加锁保护，保证只有一个线程进行缓存更新，未能获取更新锁的线程要么等待锁释放后重新读取缓存，要么返回一个空值或者默认值
    ```
- 后台更新机制
    ```
    由后台线程更新缓存， 不是由业务来更新缓存，缓存本身的有效期设置为永久，后台线程定时更新缓存
    ```
- 给缓存的失效时间加一个随机值，避免集体失效
- 使用双缓存，缓存A和B,A设置失效时间，B不设置失效
    ```
    - 从缓存A读取数据，有就返回
    - A没有数据，直接从B读取数据，直接返回，并异步启动一个更新线程
    - 更新线程同时更新A，B的缓存数据
    ```
#### 缓存穿透
请求缓存中不存在的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉

解决方案：
- 利用互斥锁，缓存失效的时候，先去获得锁，得到锁了，再去请求数据库。没得到锁，则休眠一段时间重试
- 采用异步更新策略，无论key是否取到值，都直接返回，如果查询返回的数据为空也缓存清理。value值中维护一个缓存失效时间，缓存如果过期，异步起一个线程去读数据库，更新缓存。需要做缓存预热(项目启动前，先加载缓存)操作

#### 如何解决redis的并发竞争key问题
所谓 Redis 的并发竞争 Key 的问题也就是多个系统同时对一个 Key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同

推荐方案： 分布式锁（ZooKeeper 和 Redis 都可以实现分布式锁）。（如果不存在 Redis 的并发竞争 Key 问题，不要使用分布式锁，这样会影响性能），大家去抢锁，抢到锁就做set操作即可；

#### redis和数据库双写一致性问题
首先，采取正确更新策略，先更新数据库，再删缓存。其次，因为可能存在删除缓存失败的问题，提供一个补偿措施即可，例如利用消息队列
#### 参考链接
- http://www.redis.cn/topics/data-types-intro.html#strings


