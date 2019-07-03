---
title: Scrapy Scrapy抓取本站博客
date: 2017-12-16 21:38:23
tags:
  - Scrapy
categories:
  - Scrapy
---

#### 说明
1. 通过scrapy对本站blog进行抓取
2. 抓取到的数据通过sqlalchemy操作写入mysql

#### 实现说明

- 定义表结构(models.py)

```python
# -*- coding: utf-8 -*-
import datetime
from sqlalchemy.engine.url import URL
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String, Text, DateTime, Date
from tutorial_spider.settings import MYSQL_DATABASE
 
 
def db_connect():
    return create_engine(URL(**MYSQL_DATABASE), echo=False)
 
def create_news_table(engine):
    Base.metadata.create_all(engine)
 
Base = declarative_base()
 
class TtxsgotoBlog(Base):
    __tablename__ = 'spider_ttxsgotoblog'
    id = Column(Integer, primary_key=True)
    title = Column(String(32))          # 标题
    url = Column(String(128))           # url
    publish = Column(String(32))        # 发布日期
    content = Column(Text)              # 内容
    classify = Column(String(32))       # 分类
    lable = Column(String(32))          # 标签
    create_time = Column(String(32))    # 创建时间
```

- 定义item(items.py)

```python
# -*- coding: utf-8 -*-
import scrapy
from tutorial_spider.models import TtxsgotoBlog
from scrapy.loader import ItemLoader
 
class TakeFirstItemLoader(ItemLoader):
    """
    自定义item_loader, 修改为 默认取列表中的第一个值
    """
    default_output_processor = TakeFirst()
 
 
class TtxsgotoItem(scrapy.Item):
 
    title = scrapy.Field()          # 标题
    url = scrapy.Field()            # url
    publish = scrapy.Field()        # 发布日期
    content = scrapy.Field()        # 内容
    classify = scrapy.Field(        # 分类
        output_processor=Join(',')
    )
    lable = scrapy.Field()          # 标签
    create_time = scrapy.Field()    # 创建时间
 
    def insert_to_mysql(self):
        item_sql = TtxsgotoBlog(
            title=self["title"],
            url=self["url"],
            publish=self["publish"],
            content=self["content"],
            classify=self["classify"],
            lable=self["lable"],
            create_time=self["create_time"]
        )
        return item_sql
```
- 数据处理pipeline(pipelines.py)

```python
# -*- coding: utf-8 -*-
import scrapy
from scrapy.exceptions import DropItem
from sqlalchemy.orm import sessionmaker
from tutorial_spider.models import db_connect, create_news_table
from contextlib import contextmanager
import logging
logger = logging.getLogger(__name__)
 
 
@contextmanager
def session_scope(Session):
    session = Session()
    session.expire_on_commit = False
    try:
        yield session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()
 
 
class BasicMyslqchemyPipeline(object):
    def __init__(self):
        engine = db_connect()
        create_news_table(engine)
        self.Session = sessionmaker(bind=engine)
 
    def open_spider(self, spider):
        pass
 
    def process_item(self, item, spider):
        insert_sql = item.insert_to_mysql()
        with session_scope(self.Session) as session:
            session.add(insert_sql)
        # return item
 
    def close_spider(self, spider):
        pass
 
 
class TtxsgotoFilterPipeline(object):
    """过滤某些item"""
 
    def process_item(self, item, spider):
        if item['title'] == 'Python Selenium模块':
            raise DropItem('Drop item--->', item)
        else:
            return item
 
 
class TtxsgotoBlogMysqlchemyPipeline(BasicMyslqchemyPipeline):
    """保存ttxsgoto Blog到数据库"""
    pass
```

- 抓取逻辑(spiders/ttxsgoto01.py)

```python
# -*- coding: utf-8 -*-
 
import datetime
from urllib import parse
import scrapy
import logging
 
from tutorial_spider.items import TakeFirstItemLoader, TtxsgotoItem
logger = logging.getLogger(__name__)
 
 
class Ttxsgoto01Spider(scrapy.Spider):
    name = 'ttxsgoto01'
    allowed_domains = ['ttxsgoto.github.io']
    start_urls = ['http://ttxsgoto.github.io/']
 
    custom_settings = {	# 该项目对应settings
        "ITEM_PIPELINES": {
            'tutorial_spider.pipelines.TtxsgotoFilterPipeline': 10,
            'tutorial_spider.pipelines.TtxsgotoBlogMysqlchemyPipeline': 20,
        },
    }
 
    def parse(self, response):
        articles = response.css('#main .post')
        for article in articles:
            article_url = article.css('h1 a::attr(href)').extract_first()
            url = parse.urljoin(response.url, article_url)
            yield scrapy.Request(url, callback=self.parse_article)
 
        next_url = response.css('#page-nav a[rel="next"][href]').css('::attr(href)').extract_first()
        if next_url:
            yield scrapy.Request(url=parse.urljoin(response.url, next_url), callback=self.parse)
 
    def parse_article(self, response):
        """解析文章详情"""
        item_loader = TakeFirstItemLoader(item=TtxsgotoItem(), selector=response)
        item_loader.add_css('title', '#main header a::text')
        item_loader.add_value('url', response.url)
        item_loader.add_css('publish', '.article-time time::text')
        item_loader.add_css('content', '.article-content')
        item_loader.add_css('classify', '.article-tags a::text')
        item_loader.add_css('lable', '.article-categories a::text')
        item_loader.add_value('create_time', datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
        item = item_loader.load_item()
        yield item
```

#### 查看数据
![](https://ttxsgoto.github.io/img/scrapy/selenium01.png)

#### 代码github
[Github](https://github.com/ttxsgoto/tutorial_spider)

