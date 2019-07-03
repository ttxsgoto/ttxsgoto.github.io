---
title: Python Mysqldb模块
date: 2017-03-25 21:43:12
tags:
  - Mysqldb
categories:
  - python
---
#### 说明
python中执行数据库操作，首先安装MySQL-python模块
```
yum install -y MySQL-python 或者
pip install MySQL-python
```

#### 常用操作
创建数据库连接：
```python
conn = MySQLdb.connect(
    host='127.0.0.1',
    user='test',
    passwd='test',
    db='test01',
    port=3306,
    charset=utf8)
cur = conn.cursor()        #通过获取到的数据库连接conn下的cursor()方法来创建游标，以元祖形式输出
conn.cursor(MySQLdb.cursors.DictCursor)   #字典形式输出
conn.selecct_db(dbname)  #选择数据库
cur.execute(sql, args)    #用来执行一条sql语句
cur.executemany(sql, args)    #用来执行多条sql语句
cur.fetchall()  #获取查询结果
cur.scroll(位置，模式) #调整指针
cur.rollback()  #发生错误时回滚
cur.close()  	#关闭游标
conn.commit()   #方法在提交事务，在向数据库插入一个数据时必须用此方法，否则数据不会被真正插入
conn.close()  #关闭数据库连接
```
#### 实例
```python
#!/usr/bin/env python
#coding:utf8
 
import MySQLdb
 
class Mysql(object):
 
    def __init__(self, host, port, user, passwd, db):
        self.host = host
        self.port = port
        self.user = user
        self.passwd = passwd
        self.db = db
 
        self.conn = MySQLdb.Connect(
            host = self.host,
            user = self.user,
            passwd = self.passwd,
            db = self.db,
            port = self.port,
            charset = 'utf8'
        )
        self.cur = self.conn.cursor()
 
    def select(self):
    	sql = "select * from test;"    #执行的sql语句
        try:
            self.cur.execute(sql)
            result = self.cur.fetchall()
            for line in list(result):
                print line[0],line[1]
        except Exception,e:
            print "\033[31m %s \033[0m" %e
 
    def insert(self):
        sql = """INSERT into group(id,is_admin,name,created_time)\
                VALUES（1,FALSE,'ttxsgoto','天天向上goto'),\
                (2,FALSE,'ttxsgoto02','天天向上goto02');"""
        try:
            self.cur.execute(sql) #执行一条sql语句
            self.conn.commit()	# 提交到数据库执行
        except Exception, e:
            print "\033[31m %s \033[0m" % e
 
    def manyinsert(self):
    	sql = "SELECT user_id,org_id,id from positions;"
        try:
            self.cur.execute(sql)
            result = self.cur.fetchall()
            sql2 = """ UPDATE records SET user_id=%s,org_id=%s WHERE position_id=%s ;"""
            self.cur.executemany(sql2, result)	#执行多条sql语句
            self.conn.commit()
        except Exception, e:
            print "\033[31m %s \033[0m" % e
        self.cur.close()
        self.conn.close()
 
if __name__ == '__main__':
    db = Mysql(host='127.0.0.1', port=3306, user='root', passwd='root', db='test')
    db.select()
    db.insert()
    db.manyinsert()
 
```