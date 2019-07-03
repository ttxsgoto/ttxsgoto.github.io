---
title: Elasticsearch ELK应用
date: 2017-12-26 22:46:51
tags:
  - ELK
categories:
  - Elasticsearch
---

#### ELK说明
- Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等
- Logstash是一个完全开源的工具，他可以对你的日志进行收集、分析，并将其存储供以后使用（如，搜索）
- kibana 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志

#### 功能
1.方便日志查询，统计排查问题
2.报表展示，不用登录每台服务器查看日志

#### 组件
Logstash: logstash server端用来搜集日志；

Elasticsearch: 存储各类日志；

Kibana: web化接口用作查寻和可视化日志；

搭建部署(略)

#### 应用
收集syslog,nginx access/error日志，mongo日志，程序日志，说明如下：

nignx 访问日志：因nginx访问日志可自定义，这里自定义为json格式，方便ES存储和索引
格式定义如下：
```python
log_format main_json '{ "timestamp": "$time_local", '
'"remote_addr": "$remote_addr", '
'"remote_user": "$remote_user", '
'"body_bytes_sent": "$body_bytes_sent", '
'"request_time": "$request_time", '
'"status": "$status", '
'"domain": "$host", '
'"request": "$request", '
'"request_method": "$request_method", '
'"http_referrer": "$http_referer", '
'"body_bytes_sent":"$body_bytes_sent", '
'"http_x_forwarded_for": "$http_x_forwarded_for", '
'"http_user_agent": "$http_user_agent" }';
```
其他日志收集见配置文件，说明如下：
```python
input {
    file {
        path => [ "/var/log/syslog" ]    #定义日志路径
        type => "syslog"                
        start_position => "beginning"
        ignore_older =>0
    }
    file {
        path => "/var/log/nginx/*access.log"
        codec => json
        start_position => "beginning"
        type => "nginx-acc"
    }
    file {
        path => "/var/log/nginx/*error.log"
        start_position => "beginning"
        type => "nginx-error"
        ignore_older =>0
    }
    file {
        path => [ "/data/mongo/mongo.log" ]
        type => "mongo"
        start_position => "beginning"
        #ignore_older =>0
        }
}
 
filter {
    if [type] == "syslog" {
        grok {    #grok 功能将字符串转换为相应的字段，方便检索
            match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
            add_field => [ "received_at", "%{@timestamp}" ]
            add_field => [ "received_from", "%{host}" ]
            }    
            date {
                    match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
            }
        }
         
    if [type] == "mongo" {
                grok {
#mongo_v3                        match => ["message","%{TIMESTAMP_ISO8601:timestamp}\s+%{MONGO3_SEVERITY:severity}\s+%{MONGO3_COMPONENT:component}\s+(?:\[%{DATA:context}\])?\s+%{GREEDYDATA:body}"]
            match => ["message","%{SYSLOGTIMESTAMP:timestamp} \[%{WORD:component}\] %{GREEDYDATA:body}"]     #mongo_v2
               }
                if[body]=~"ms$" {
                                        grok {
                                                match => ["body","query\s+%{WORD:db_name}\.%{WORD:collection_name}.*}.*\}(\s+%{NUMBER:spend_time:int}ms$)?"]
                                        }
                                }
                date {
                        match => [ "timestamp", "UNIX", "YYYY-MM-dd HH:mm:ss", "ISO8601" ]
                        remove_field => ["timestamp"]
                }
        }
    if [type] == "nginx-error" {
                grok {
                        match => { "message" => "(?<timestamp>%{YEAR}[./-]%{MONTHNUM}[./-]%{MONTHDAY}[- ]%{TIME}) \[%{LOGLEVEL:severity}\] %{POSINT:pid}#%{NUMBER}: %{GREEDYDATA:errormessage}(?:, client: (?<client>%{IP}|%{HOSTNAME}))(?:, server: %{IPORHOST:server})(?:, request: %{QS:request})?(?:, upstream: \"%{URI:upstream}\")?(?:, host: %{QS:host})?(?: referrer: \"%{URI:referrer}|-\")?" }
                        overwrite => [ "message" ]
                }
                date {
                        match => [ "nginx_error_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
                        remove_field => [ "timestamp" ]
                }
        }
}
output {
    if [type] == "nginx-acc" {
        elasticsearch {    #存储
            hosts => ["127.0.0.1:9200"]
            index => "nginx_access-%{+YYYY.MM.dd}"
        }        
    }
    if [type] == "nginx-error" {
        elasticsearch {
            hosts => ["127.0.0.1:9200"]
            index => "nginx_error-%{+YYYY.MM.dd}"
        }        
    }
    if [type] == "syslog" {
        elasticsearch {
            hosts => ["127.0.0.1:9200"]
            index => "syslog-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "mongo" {
                elasticsearch {
            hosts => ["127.0.0.1:9200"]
                        index => "mongo-%{+YYYY.MM.dd}"
                }
        }
}
```
采集到数据展示如下：
![elk](https://ttxsgoto.github.io/img/elk/elk.png)
