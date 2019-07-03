---
title: 运维 Mongo日常笔记
date: 2017-02-16 21:57:18
tags:
  - Mongo
categories:
  - 运维
---

### 认证授权相关
帐号是跟着库走的，所以在指定库里授权，必须也在指定库里验证(auth)，然后在切换到对应的库中

**认证登录说明**
超级用户：拥有最大权限，存储在admin数据库中，
数据库用户：存储在单个数据库中，最能访问对应的数据库
用户信息保存在db.system.users中

**开启认证方法**
- 启动添加： --auth
- 配置文件：security.authorization: enabled

**用户和权限的特性**
1. 数据库是由超级用户来创建的，一个数据库可以包含多个用户，一个用户只能在一个数据库下，不同数据库中的用户可以同名
2. 如果在 admin 数据库中不存在用户，即使 mongod 启动时添加了 –auth 参数，此时不进行任何认证还是可以做任何操作
3. 在 admin 数据库创建的用户具有超级权限，可以对 MongoDB 系统内的任何数据库的数据对象进行操作
4. 特定数据库比如 test1 下的用户 test_user1，不能够访问其他数据库 test2，但是可以访问本数据库下其他用户创建的数据
5. 不同数据库中同名的用户不能够登录其他数据库。比如数据库 test1 和 test2 都有用户 test_user，以 test_user 登录 test1 后,不能够登录到 test2 进行数据库操作

#### 授权命令
```
use xxx;						#进入某库，创建某库
db.createUser({user: "ttxsgoto",pwd:"ttxsgoto",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
db.createUser({user: "ttxsgoto",pwd:"ttxsgoto",roles:[{role:"read",db:"test"}]})	        #读
db.createUser({user: "ttxsgoto01",pwd:"ttxsgoto01",roles:[{role:"readWrite",db:"test"}]})	#读写
db.createUser({user: "ttxsgoto02",pwd:"ttxsgoto02",roles:[{role:"root",db:"test"}]})	        #超级root权限
db.system.users.remove({user:"ttxsgoto"})	#删除用户
db.changeUserPassword('ttxsgoto','test'); 	#修改密码的方法
```
**具体权限说明**
```
Built-In Roles（内置角色）：
    1. 数据库用户角色：read、readWrite;
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
    4. 备份恢复角色：backup、restore；
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
    6. 超级用户角色：root
    7. 内部角色：__system

具体角色：
Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```

```
# 在admin库中
db.createUser({user: "ttxsgoto02",pwd:"ttxsgoto02",roles:[{role:"readWrite",db:"test01"},{role:"readWrite",db:"test02"}]})	#在admin库下创建test01，test02库的账号

在这种情况下，不能直接在对应的库中登录验证，而应该在对应的这个库中进行认证后，在切换到其他库中
use admin
db.auth('ttxsgoto02','ttxsgoto02')
use test02；
```
#### 登录验证
```
方法一：
mongo
use admin
db.auth("admin","abc")
方法二：
mongo -u admin -p admin --authenticationDatabase admin
```
### 日常使用
#### 常用命令
```
mongo 127.0.0.1/admin -uadmin -p'admin'		#连接mongo数据库
use admin;			#进入数据库
show users;			#查看当前库下的用户
show collections/tables;        #查看当前库中的collections
db.getCollectionNames();		#得到当前db的所有集合
db.auth('ttxsgoto','ttxsgoto')	#登录验证
db.getName()			#查看当前使用的数据库
db.stats()			#查看当前db状态
db.getMongo();			#查看当前db连接机器的地址
db.abc.find();	                #查看集合中的所有数据
db 		                #查看当前所在的数据库
db.system.users.find().pretty()	#在admin库中查看所有账号
db.getMongo();			#查看当前db连接机器的地址
db.repairDatabase()		#修复当前数据库
db.getCollectionNames()		#得到当前db的所有集合
db.printCollectionStats()	#显示当前db所有聚集索引的状态
```
#### 增删改查
```
# 插入集合
content={name:"xxxx",sex:"m"}
db.collections.insert(content)
db.createCollection("collName", {size: 20, capped: 5, max: 100})	#创建一个集合
db.getCollection("account")		#得到集合名称
# 查询：
db.inventory.find();
db.foo.find()		#对当前数据库中的foo集合进行数据查找，所有数据
db.foo.find({a:1})	#对当前数据库中的foo集合中条件包含a=1的项进行查询
# 删除：
db.dropDatabase()	#删除当前使用的数据库
```
#### 备份相关
```
mongoimport(导入)/mongoexport(导出)：(将collection导出为json格式或csv格式)
mongoexport -d test -c t1 -o t1.dat	#导出json格式
-c	#指明导出的集合
-d	#使用库
mongoexport -d test -c t1 -csv -f num -o t1.dat	#导出csv格式
-csv	#指明导出csv格式
-f	#指明需要导出哪些列
mongoimport -d test -c t1 -file t1.dat				#还原
mongoimport -d test -c t1 -type csv --headerline -file t1.dat	#还原csv格式的数据
--headerline	#指明不导入第一行，因为第一行为列名
mongodump/mongorestore:（整库备份还原,库级别操作,先执行查询动作然后把所有查询结果写入到硬盘中，但在内存中的数据未写入磁盘中）
mongodump -h x.x.x.x  --port  27017 -uroot -p xxx  -d test -o /bak/mongodump	#导出
mongodump -h x.x.x.x  --port  27017 -uroot -p xxx	-o /bak/allmongobak	#导出所有
-h ip
--port port
-u user
-p password
-d database
-c collection
-o outfile
mongorestore -h IP  --port -u user -p password 	-d test  --drop	/bak/mongodump/*	#恢复
--drop	#恢复前先删除所有记录
--noIndexRestore	#不还原索引
例子说明
# 备份
mongoexport -u root -p root --host=127.0.0.1:27017 --authenticationDatabase admin -d database -c collection -o collection.json

mongodump -u root -p root --host=127.0.0.1:27017 --authenticationDatabase admin -d database -c collection -o 1202.dat
# 还原
mongorestore -u root -p root --authenticationDatabase admin -d database_20161202 --noIndexRestore --drop 1202.dat/collection
mongoimport -u root -p root --host=127.0.0.1:27017 --authenticationDatabase admin -d database_20161202 -c collection  --file  collection.json
```
**库表级备份还原的区别:**
mongorestore和mongodump提供的是对mongo数据库的整个数据库的恢复和备份，而mongoimport和mongoexport则是提供更细粒度的collection级别的数据导入和导出。两者的粒度不同，mongoimport和mongoexport粒度更细，相对来说，更加灵活。其次，mongoimport和mongoexport只是将集合中的数据导出和导入，但是没有对数据库中的其它成分进行备份（比如索引），而mongorestore和mongodump则是对数据库中的所有成分（包括索引等其它）进行恢复和备份。然而，这也导致了mongorestore和mongodump导出的文件比较大耗时较长，而mongoimport和mongoexport导出的文件比较小，速度比较快，而且格式较为灵活。




