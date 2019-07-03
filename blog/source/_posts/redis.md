---
title: 运维 Redis日常笔记
date: 2017-02-18 22:16:52
tags:
  - Redis
categories:
  - 运维
---
### redis设置密码
```
方法一（命令行）：
CONFIG SET requirepass  password	#设置密码为password
auth password	#登录
ping	#验证
CONFIG SET requirepass  ''	#取消密码，设置为空
redis-cli -h 127.0.0.1 -p 6379 -a "ttxsgoto" 	#验证连接
 
方法二（配置文件）：
/etc/redis.conf中
#requirepass foobared  
去掉行前的注释，并修改密码为所需的密码,保存文件
重启redis server完成
 
#导出
redis-cli -h 127.0.0.1 -p 6379 -a "ttxsgoto" keys xxx* > xxx.txt
```
### 常用命令
```
keys *  #查看所有的keys
get key #查看key
set key vlaue   #设置key
del key #删除key
 
#其他   http://redisdoc.com/
dbsize  #当前数据库key数量
select 1    #切换到1号数据库
config get *    #返回配置参数的变量和值
info [Memory]    #返回redis服务器的各种信息和统计数据，如版本，内存使用情况  http://redisdoc.com/server/info.html
monitor    #实时打印出 Redis 服务器接收到的命令，调试用
bgsave  #fork出一个新子进程，负责将数据保存到磁盘中
slowlog get num    #查看slowlog http://redisdoc.com/server/slowlog.html
slowlog len/reset     #slowlog数量/清空
save    #备份
move key db-index   # 返回1成功，0 如果key不存在，或者已经在指定数据库中
 
#删除所有key
flushdb     #删除当前数据库中的所有Key
flushall    #删除所有数据库中的所有key
 
查看建是否存在
exists key   有返回(integer) 1，没有返回(integer) 0
 
rename key newkey   #更改键的名称
type key    #返回键的数据类型
```
### redis备份&恢复
```
######## 备份 ########
save    #备份，写入rdb文件
bgsave  #fork出一个新子进程，负责将数据保存到磁盘中
 
######## 恢复 ########
Redis 恢复的机制
    - 如果只配置 AOF ，重启时加载 AOF 文件恢复数据；
    - 如果同时配置了 RDB 和 AOF ，启动是只加载 AOF 文件恢复数据；
    - 如果只配置 RDB，启动是将加载 dump 文件恢复数据
 
1.aof 中恢复数据(配置如下)
    appendonly yes
    dir /data/redis/
2.rdb 中恢复数据(配置如下)
    appendonly no
    dir /data/redis/
3.启动服务
 
#####重启服务时，会自动加载备份文件中的数据，但如果密码没有写入配置问题，会丢失需要重新设置
#定时备份文件
对于RDB和AOF，都是直接拷贝文件即可，可以设定crontab进行定时备份： cp /var/lib/redis/dump.rdb /somewhere/safe/dump.$(date +%Y%m%d%H%M).rdb
 
检查修复AOF文件：
redis-check-aof data/appendonly.aof
 
# 数据快照，备份，主从
http://blog.csdn.net/zhu_xun/article/details/16806697
```
### 配置文件
```
#redis.conf
################################## INCLUDES ###################################
# 其他个性化设置
# include /path/to/local.conf
# include /path/to/other.conf
 
################################ GENERAL  #####################################常用
daemonize no    #默认不为守护进程运行，设置为yes修改为守护进程
pidfile /var/run/redis.pid  #如果为守护进程模式，pid文件
port 6379   #监听端口
tcp-backlog 511     # TCP 监听的最大容纳数量,在高并发的环境下，你需要把这个值调高以避免客户端连接缓慢的问题,/proc/sys/net/core/somaxconn 和设置值 相同
# bind 192.168.1.100 10.0.0.1   #你如果只想让它在一个网络接口上监听，那你就绑定一个IP或者多个IP，多个IP用空格隔开
timeout 0   # 指定在一个 client 空闲多少秒之后关闭连接（0 就是不管它）
tcp-keepalive 0 # tcp 心跳包，推荐一个合理的值就是60秒
loglevel notice     # 定义日志级别，notice (适用于生产环境)
logfile ""      # 指定日志文件的位置
 
# 要想把日志记录到系统日志，就把它改成 yes，
# 也可以可选择性的更新其他的syslog 参数以达到你的要求
# syslog-enabled no
 
# 设置 syslog 的 identity。
# syslog-ident redis
 
# 设置 syslog 的 facility，必须是 USER 或者是 LOCAL0-LOCAL7 之间的值。
# syslog-facility local0
 
databases 32    #设置数据库的数目，启动32个数据库，默认为16个(0-15)
 
################################ SNAPSHOTTING  ################################快照
# 存 DB 到磁盘：
#   格式：save <间隔时间（秒）> <写入次数>
#   根据给定的时间间隔和写入次数将数据保存到磁盘
#
#   下面的例子的意思是：
#   900 秒内如果至少有 1 个 key 的值变化，则保存
#   300 秒内如果至少有 10 个 key 的值变化，则保存
#   60 秒内如果至少有 10000 个 key 的值变化，则保存
#　　
#   注意：你可以注释掉所有的 save 行来停用保存功能。
#   也可以直接一个空字符串来实现停用：
#   save ""
 
save 900 1
save 300 10
save 60 10000
 
# 默认情况下，如果 redis 最后一次的后台保存失败，redis 将停止接受写操作，
# 这样以一种强硬的方式让用户知道数据不能正确的持久化到磁盘，
# 否则就会没人注意到灾难的发生。
#
# 如果后台保存进程重新启动工作了，redis 也将自动的允许写操作。
#
# 你可能不希望 redis 这样做，那你就改成 no 
stop-writes-on-bgsave-error yes
 
# 是否在 dump .rdb 数据库的时候使用 LZF 压缩字符串
# 默认都设为 yes
# 如果你希望保存子进程节省点 cpu ，你就设置它为 no ，
# 不过这个数据集可能就会比较大
rdbcompression yes
 
# 是否校验rdb文件
rdbchecksum yes
 
# 设置 dump 的文件位置
dbfilename dump.rdb 
 
# 工作目录
# 例如上面的 dbfilename 只指定了文件名，
# 但是它会写入到这个目录下。这个配置项一定是个目录，而不能是文件名。
dir ./      #设置到 /etc/redis下 dir '/etc/redis/'
 
################################# REPLICATION #################################主从复制
#######
 
################################## SECURITY ###################################安全
# 设置认证密码
requirepass foobared
 
################################### LIMITS ####################################限制
# 一旦达到最大限制，redis 将关闭所有的新连接
# 并发送一个‘max number of clients reached’的错误
# maxclients 10000
 
# 最大使用内存
# maxmemory <bytes>
 
# 最大内存策略，你有 5 个选择。
# 
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# volatile-lru -> 使用 LRU 算法移除包含过期设置的 key 。
# allkeys-lru -> remove any key accordingly to the LRU algorithm
# allkeys-lru -> 根据 LRU 算法移除所有的 key 。
# volatile-random -> remove a random key with an expire set
# allkeys-random -> remove a random key, any key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# noeviction -> don't expire at all, just return an error on write operations
# noeviction -> 不让任何 key 过期，只是给写入操作返回一个错误
 
############################## APPEND ONLY MODE ###############################
 
appendonly no   #在启动时Redis会逐个执行AOF文件中的命令来将硬盘中的数据载入到内存中，载入的速度相较RDB会慢一些
appendfilename "appendonly.aof"
 
# appendfsync always
appendfsync everysec
no-appendfsync-on-rewrite no
 
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
 
################################ LUA SCRIPTING  ###############################
lua-time-limit 5000
 
################################ REDIS CLUSTER  ###############################集群
# cluster-enabled yes   # 启用或停用集群
 
# cluster-config-file nodes-6379.conf
# cluster-node-timeout 15000
# cluster-slave-validity-factor 10
# cluster-migration-barrier 1
 
################################## SLOW LOG ###################################慢日志
 
slowlog-log-slower-than 10000   # 慢查询记录时间10000ms
slowlog-max-len 128             # 记录数据条数 
 
############################# Event notification ##############################
notify-keyspace-events ""
 
############################### ADVANCED CONFIG ###############################
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
 
list-max-ziplist-entries 512
list-max-ziplist-value 64
 
set-max-intset-entries 512
 
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
 
hll-sparse-max-bytes 3000
 
activerehashing yes
 
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
 
hz 10
 
aof-rewrite-incremental-fsync yes
```