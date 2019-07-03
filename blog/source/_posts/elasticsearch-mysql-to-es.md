---
title: Elasticsearch Logstash-input-jdbc同步mysql数据到ES
date: 2017-12-27 21:08:17
tags:
  - Logstash-input-jdbc
categories:
  - Elasticsearch
---
#### 说明
将Mysql中的数据同步到ES集群中，方便搜索功能，通过logstash自带的插件logstash-input-jdbc来轻松完成

#### 安装
```
./logstash-plugin install  logstash-input-jdbc
```

#### 配置文件
jdbc.conf
```python
input {
    stdin {
    }
    jdbc {
      jdbc_connection_string => "jdbc:mysql://127.0.0.1:3307/test"
      jdbc_user => "root"
      jdbc_password => "root"
      # jdbc driver
      jdbc_driver_library => "/data/es/mysql-connector-java-5.1.39.jar"
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      # 执行的sql的路径
      statement_filepath => "sql.sql"
      # 需要导入的sql语句查出来的数据
      #statement => "SELECT * FROM zhihuquestion"
      # 定时字段 各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
      schedule => "* * * * *"
      #schedule => "*/2 * * * *"    # 每2分钟更新一次
      type => "test_index"
    }
}
 
filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}
 
output {
    elasticsearch {
        hosts => ["http://127.0.0.1:9200"]
        index => "test_index"
        document_type => "test"
        #自增ID编号
        document_id => "%{id}"
    }
    # 以json格式输出
    stdout {
        codec => json_lines
    }
}
```
#### 启动
```
../bin/logstash -f jdbc.conf
```
#### 参考文档
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html
https://qbox.io/blog/migrating-mysql-data-into-elasticsearch-using-logstash
http://blog.csdn.net/laoyang360/article/details/51747266
