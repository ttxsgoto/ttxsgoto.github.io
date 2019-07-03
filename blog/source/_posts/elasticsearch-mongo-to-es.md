---
title: Elasticsearch Mongo-connector同步mongo数据到ES
date: 2017-12-27 21:16:47
tags:
  - Mongo-connector
categories:
  - Elasticsearch
---
#### 说明
1. 通过mongo-connector将mongo数据实时同步到es中
2. mongo运行在replica-set模式，同时需要elastic2_doc_manager将数据写入ES
3. 开启replica-set模式后，写入mongo的数据都可以同步到es，即使当前mongo-connector没有启动，启动后也会将数据写入es中

#### 安装
```python
pip install mongo-connector
pip install 'elastic2-doc-manager[elastic5]'    # ES6.1版本也可使用
```

#### mongod 以replica-set模式运行
- 服务端:mongod version为3.6.0
```python
./bin/mongod --replSet "rs0" --dbpath /data/tools/mongodb-3.6.0/data --port 27018 --bind_ip 0.0.0.0
```
- 客户端设置
```python
# 连接mongo
./bin/mongo --host 127.0.0.1 --port 27018
# 配置复制集
rs.initiate()   # 初始化副本集
rs.conf()       # 验证副本集的配置
rs.status()     # 验证副本集的状态
```

#### mongo-connector启动

mongo-connector -m 127.0.0.1:27018 -t 127.0.0.1:9200 -d elastic2_doc_manager

#### 数据测试
- 新增数据
```python
use ttxsgoto
db.test.insert({name:"ttxsgoto01", sex:"m", project:"python"})
db.test.insert({name:"ttxsgoto02", sex:"m", project:"java"})
 
# mongo查看数据
db.test.find()
 
# ES查看数据
{
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "ttxsgoto",
        "_type": "test",
        "_id": "5a42188f1db5d952cbcea0ef",
        "_score": 1,
        "_source": {
          "project": "java",
          "name": "ttxsgoto02",
          "sex": "m"
        }
      },
      {
        "_index": "ttxsgoto",
        "_type": "test",
        "_id": "5a4218501db5d952cbcea0ee",
        "_score": 1,
        "_source": {
          "project": "python",
          "name": "ttxsgoto01",
          "sex": "m"
        }
      }
    ]
  }
}
```
- 修改数据
```python
db.test.update({'name':'ttxsgoto01'}, {$set:{'name':'ttxs'}})
 
# mongo查看数据
db.test.find()
 
# ES查看数据
{
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "ttxsgoto",
        "_type": "test",
        "_id": "5a42188f1db5d952cbcea0ef",
        "_score": 1,
        "_source": {
          "project": "java",
          "name": "ttxsgoto02",
          "sex": "m"
        }
      },
      {
        "_index": "ttxsgoto",
        "_type": "test",
        "_id": "5a4218501db5d952cbcea0ee",
        "_score": 1,
        "_source": {
          "project": "python",
          "name": "ttxs",
          "sex": "m"
        }
      }
    ]
  }
}
```
- 删除数据
```python
db.test.remove({'name':'ttxsgoto02'})
 
# ES查看数据
{
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "ttxsgoto",
        "_type": "test",
        "_id": "5a4218501db5d952cbcea0ee",
        "_score": 1,
        "_source": {
          "project": "python",
          "name": "ttxs",
          "sex": "m"
        }
      }
    ]
  }
}
```
- 删除db
```python
db.dropDatabase() # 删除数据库后，ES中对应的索引也被删除
```
#### 参考文档
https://docs.mongodb.com/manual/tutorial/deploy-replica-set/
https://github.com/mongodb-labs/elastic2-doc-manager
http://blog.csdn.net/laoyang360/article/details/51842822
