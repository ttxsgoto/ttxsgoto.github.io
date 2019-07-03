---
title: Elasticsearch Elasticsearch_dsl 基本查询
date: 2017-12-24 15:25:33
tags:
  - Elasticsearch_dsl
categories:
  - Elasticsearch
---
#### 说明
 通过elasticsearch_dsl提供的API操作es进行基本查询操作
 
#### 创建mapping
```python
from elasticsearch_dsl import DocType, Date, Keyword, Text, Integer
from elasticsearch_dsl.connections import connections
 
connections.create_connection(hosts=["localhost"])
# connections.create_connection(hosts=["http://admin:admin@127.0.0.1:8080"])    # 使用认证连接
 
 
class ZhihuQ(DocType):
    id = Integer()
    zhihu_id = Integer()                    # 知乎idlong
    topics = Keyword()                      # 主题text
    url = Keyword()                         # urlkeyword
    title = Text(analyzer="ik_max_word")    # 标题text
    content = Text(analyzer="ik_max_word")  # 内容text
    answer_num = Integer()                  # 回答数量int
    comments_num = Integer()                # 评论数量int
    watch_user_num = Integer()              # 关注者数量int
    click_num = Integer()                   # 浏览数量int
    crawl_time = Date(format='date_optional_time||yyyy-MM-dd HH:mm:ss') # 创建时间
 
    class Meta:
        index = "zhihuquestion"
        doc_type = "question"
 
 
if __name__ == "__main__":
    ZhihuQ.init()   # 初始化mapping
```

#### 基本查询

##### 查询所有，指定返回数量,设置分页

```python
GET zhihuquestion/_search
{
  "query": {
    "match_all": {}
  },
   "from": 0,
   "size": 5
}
# python：
search = ZhihuQ.search()
result = search.query().extra(size=1000)[0:5]
data = result.execute()
data = data.to_dict()['hits']['hits']

```
##### match查询
使用分词处理后查询
```python
GET zhihuquestion/_search
{
    "query": {
        "match": {
            "title": "python"
        }
    }
}
# python
result = search.query("match", title='Python')
data = result.execute()
data = data.to_dict()['hits']['hits']

```
##### 多字段查询
```python
GET zhihuquestion/_search
{
    "query": {
        "multi_match": {
            "query": "python",
            "fields": [
                "title",
                "content"
            ]
        }
    }
}
# python
multi_match = MultiMatch(query='python', fields=['title', 'content'])
q = search.query(multi_match)
result = q.query()
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### term查询
值不做解析处理，直接查询,完全匹配
```python
GET zhihuquestion/_search
{
    "query": {
        "term": {
            "topics": "python"
        }
    }
}
# python
result = search.query('term', title='Python')
data = result.execute()
data = data.to_dict()['hits']['hits']

```
##### terms查询(多词条)
任何一个满足都可以返回数据
```python
GET zhihuquestion/_search
{
    "query": {
        "terms": {
            "topics": [
                "python",
                "Python"
            ]
        }
    }
}
# python
result = search.query('terms', topics=['Python', 'python'])
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### 词条(Term)查询-排序(Sorted)
指定返回字段
```python
GET zhihuquestion/_search
{
    "sort": [
        {
            "crawl_time": {
                "order": "asc"
            }
        }
    ],
    "query": {
        "terms": {
            "title": [
                "Python",
                "python"
            ]
        }
    },
    "_source": [
        "crawl_time"
    ]
}
# python
result = search.query('terms', title=['Python', 'python']).source(['crawl_time']).sort({'crawl_time':{"order" : "asc",}})
data = result.execute()
data = data.to_dict()['hits']['hits']

```
##### range 范围查询
用于日期、数字和字符串类型的字段
```python
GET zhihuquestion/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "title": "Python"
                    }
                },
                {
                    "range": {
                        "crawl_time": {
                            "gte": "2017-12-21 19:19:44",
                            "lte": "now"
                        }
                    }
                }
            ]
        }
    }
}
# python
result = search.query("match", title='Python').query("range", crawl_time={'gte':'2017-12-21 19:19:44','lte': 'now'})
data = result.execute()
data = data.to_dict()['hits']['hits']
```

##### bool查询

- must 等同于 AND
- must_not 等同于 NOT
- should 等同于 OR

```python
GET zhihuquestion/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "title": "python"
                    }
                },
                {
                    "match": {
                        "title": "c"
                    }
                }
            ]
        }
    }
}
# python
q = Q("match", title='python') & Q("match", title='c')
q = Q("match", title='python') | Q("match", title='c')
q = ~Q("match", title='python') & ~Q("match", title='c')
result = search.query(q)
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### highlighting高亮
```python
GET zhihuquestion/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "title": "python"
                    }
                },
                {
                    "match": {
                        "title": "c"
                    }
                }
            ]
        }
    },
    "highlight": {
        "fields": {
            "title": {
                "fragment_size": 50
            }
        }
    }
}
# python
q = Q("match", title='python') & Q("match", title='c')
s = search.query(q)
result = s.highlight('title', fragment_size=50)
data = result.execute()
data = data.to_dict()['hits']['hits']
```

##### 模糊(fuzzy)查询
在 Match检索 和多匹配检索中可以启用模糊匹配来捕捉拼写错误;模糊度是基于原始单词的编辑距离来指定,当术语长度大于 5 个字符时，AUTO 的模糊值等同于指定值 “2”。但是，80％ 拼写错误的编辑距离为 1，所以，将模糊值设置为 1 可能会提高您的整体搜索性能
```python
GET zhihuquestion/_search
{
    "query": {
        "multi_match": {
            "fields": [
                "title",
                "content"
            ],
            "fuzziness": "AUTO",
            "query": "Python"
        }
    }
}
# python
multi_match = MultiMatch(query='Python', fields=['title', 'content'], fuzziness='AUTO')
q = search.query(multi_match)
result = q.query()
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### 通配符(wildcard)查询 

通配符查询允许指定匹配的模式，而不是整个词组（term）检索

？ 匹配任何字符, * 匹配零个或多个字符
```python
GET zhihuquestion/_search
{
    "query": {
        "wildcard": {
            "title": "pyth*"
        }
    }
}
# python
result = search.query('wildcard', title='python*')
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### match_phrase查询(短语查询)
匹配短语查询要求查询字符串中的所有词都存在于文档中，按照查询字符串中指定的顺序并且彼此靠近;默认情况下，这些词必须完全相邻，但可以指定偏离值（slop value)，该值指示在仍然考虑文档匹配的情况下词与词之间的偏离值。
```python
GET zhihuquestion/_search
{
    "query": {
        "multi_match": {
            "fields": [
                "title"
            ],
            "type": "phrase",
            "slop": 6,
            "query": "python下载"
        }
    }
}
# python
multi_match = MultiMatch(query='python下载', fields=['title'],
                                 type='phrase',
                                 slop=4
                                 )
q = search.query(multi_match)
result = q.query()
data = result.execute()
data = data.to_dict()['hits']['hits']
```
##### 短语前缀(Match Phrase Prefix)查询
匹配词组前缀查询在查询时提供搜索即时类型或“相对简单”的自动完成版本，而无需以任何方式准备数据
```python
GET zhihuquestion/_search
{
    "query": {
        "multi_match": {
            "fields": [
                "title"
            ],
            "type": "phrase_prefix",
            "slop": 2,
            "query": "python精通"
        }
    }
}
# python
multi_match = MultiMatch(query='python精通', fields=['title'],
                                 type='phrase_prefix',
                                 slop=2
                                 )
q = search.query(multi_match).source(['title'])
result = q.query()
data = result.execute()
data = data.to_dict()['hits']['hits']
```

##### aggregation聚合
聚合类型: Bucketing, Metric, Matrix, Pipeline
```python
# metric 计算相关, max, min, avg等
GET zhihuquestion/_search
{
    "query": {
        "match_all": {}
    },
    "aggs": {
        "max_click_num": {
            "max": {
                "field": "click_num"
            }
        }
    }
}
# python
_agg = A('max', field='click_num')
result = search.aggs.metric('max_click_num', _agg)
data = result.execute()
print data.aggregations.max_click_num   # 得到点击量最大的值
 
# bucket
_agg = A('terms', field='comment')
f = search.aggs.bucket('bucket_comment', _agg)
query_word = json.dumps(f.to_dict())
print query_word
response = search.execute()
print response.aggregations.bucket_comment.buckets
print response.to_dict()
```




