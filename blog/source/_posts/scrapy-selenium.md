---
title: Scrapy Scrapy+Selenium抓取本站博客
date: 2017-12-17 21:12:00
tags:
  - Selenium
  - Scrapy
categories:
  - Scrapy
---

#### 说明
1. 通过scrapy+selenium对本站blog进行抓取
2. 抓取到的数据通过sqlalchemy操作写入mysql
3. 用来练习scrapy+selenium模拟操作浏览器，没有对blog正文进行相应处理

#### 实现说明

- 定义表结构(models.py)

```python
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
import scrapy
from tutorial_spider.models import TtxsgotoBlog
 
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
 
 
class TtxsgotoBlogMysqlchemyPipeline(BasicMyslqchemyPipeline):
    """保存ttxsgoto Blog到数据库"""
    pass
```

- 抓取逻辑(spiders/ttxsgoto.py)

```python
# -*- coding: utf-8 -*-
"""
    - scrapy + selenium 爬取ttxsgoto.github.io 文章
    - 主要用来练习scrapy和selenium结合完成抓取工作
    - 使用过程非标准scrapy框架
"""
import datetime
import logging
from urllib import parse
import scrapy
import time
from scrapy.http import HtmlResponse
from selenium import webdriver
from scrapy.xlib.pydispatch import dispatcher
from scrapy import signals
from tutorial_spider.items import TtxsgotoItem
from tutorial_spider.pipelines import TtxsgotoBlogMysqlchemyPipeline
 
logger = logging.getLogger(__name__)
 
 
class TtxsgotoSpider(scrapy.Spider):
    name = 'ttxsgoto'
    allowed_domains = ['ttxsgoto.github.io']
    start_urls = ['http://ttxsgoto.github.io/']
 
    def __init__(self, *args, **kwargs):
        self.driver = webdriver.Chrome()
        self.driver.maximize_window()
        logger.info("开始爬取ttxsgoto.github.io数据")
        super(TtxsgotoSpider, self).__init__(*args, **kwargs)
        dispatcher.connect(self.close_driver, signals.spider_closed)
 
    def close_driver(self, spider):
        '''
       关闭浏览器
        '''
        self.driver.quit()
 
    def start_requests(self):
        self.driver.get(self.start_urls[0])
        res = HtmlResponse(url='index html', body=self.driver.page_source, encoding="utf-8")
        title_text = res.css('#main section h1 a::text')[0].root.strip()
        self.driver.find_element_by_link_text(title_text).click()    # 点击进去
        time.sleep(2)
        if title_text in self.driver.page_source:
            self.detail_parse(self.driver.page_source, title_text)
 
        while True:
            try:
                key_word = self.driver.find_elements_by_class_name("next")[0].text  # 进行下一篇文章抓取
            except (TypeError,IndexError):
                self.driver.quit()
                key_word = None
            if not key_word:
                break
            self.driver.find_element_by_link_text(key_word).click()
            time.sleep(2)
            res = HtmlResponse(url='next html', body=self.driver.page_source, encoding="utf-8")
            title_text = res.css('.article-info h1 a::text').extract_first()
            self.detail_parse(self.driver.page_source, title_text)
 
    def detail_parse(self, page_source, title):
        res = HtmlResponse(url='detail html', body=page_source, encoding="utf-8")
        item = TtxsgotoItem()
        item['title'] = title   # 标题
        _url = res.css('#main h1 a::attr(href)').extract_first()
        item['url'] = parse.urljoin(self.start_urls[0], _url)   # url
        item['publish'] = res.css('.article-time time::text').extract_first()   # 发布日期
        item['content'] = res.css('.article-content').extract_first()   # 内容
        classify_list = res.css('.article-tags a::text').extract()
        item['classify'] = ','.join(classify_list)  # 分类
        item['lable'] = res.css('.article-categories a::text').extract_first()   # 标签
        item['create_time'] = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S') # 创建日期
        pipeline = TtxsgotoBlogMysqlchemyPipeline()
        pipeline.process_item(item, self.name)	# 写入数据库
```

#### 查看数据
![](https://ttxsgoto.github.io/img/scrapy/selenium01.png)

#### 代码github
[Github](https://github.com/ttxsgoto/tutorial_spider)
