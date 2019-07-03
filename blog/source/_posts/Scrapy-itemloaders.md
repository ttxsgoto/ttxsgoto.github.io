---
title: Scrapy Itemloaders
date: 2017-12-02 22:01:14
tags:
  - Itemloaders
categories:
  - Scrapy
---

### Item Loaders
Item Loaders 被设计用来提供一个既弹性又高效简便的构件， 以扩展或重写爬虫或源格式(HTML, XML之类的)等区域的解析规则


### 使用item loader填充item
- add_xpath  # 通过xpath选取数据
- add_css    # 通过css选取数据
- add_value  # 通过value得到数据

```python
item = ItemLoader(item=Movietop250Item(), response=response)
 
item.add_css('num', "node.css('em::text')")
item.add_css('movie_detail_url', '.hd a::attr(href)')
item.add_css('img_url', 'a img::attr(src)')
item.add_css('name', 'a img::attr(src)')
item.add_css('grade', '.rating_num::text')
item.add_css('comment', '.star span::text')
item.add_css('info', '.inq::text')

load_item = item.load_item()    # load_item() 方法,返回通过调用 add_xpath(), add_css(), and add_value() 所提取和收集到的数据的Item.
yield load_item
```

### 输入/输出处理器
- Item Loader每个字段中都包含一个输入处理器和一个输出处理器
- input_processor输入处理器：收到数据时立刻提取数据 (通过 add_xpath(), add_css() 或者 add_value() 方法) 之后输入处理器的结果被收集起来并且保存在ItemLoader内(但未分配给该Item)
- output_processor输出处理器：收集所有的数据后, 调用 ItemLoader.load_item()得到Item 对象。在这步中先调用输出处理器来处理之前收集到的数据，然后再存入Item中。输出处理器的结果是被分配到Item的最终值

### Items Loaders
Item Loaders 的声明类似于Items，以class的语法来声明：
```python
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join
  
class ProductLoader(ItemLoader):
    default_output_processor = TakeFirst()
    name_in = MapCompose(unicode.title)
    name_out = Join()
    price_in = MapCompose(unicode.strip)
 
    # ...
```
input processors 以_in为后缀来声明，output processors 以_out 为后缀来声明。也可以用ItemLoader.default_input_processor 和ItemLoader.default_output_processor 属性来声明默认的 input/output processors

### Input and Output Processors

在定义Item的时候声明输入输出处理器
```python
import scrapy
from scrapy.loader.processors import Join, MapCompose, TakeFirst
from w3lib.html import remove_tags
 
def filter_price(value):
    """定义方法来处理值"""
    if value.isdigit():
        return value
 
class Product(scrapy.Item):
    name = scrapy.Field(
        input_processor=MapCompose(remove_tags),
        output_processor=Join(),
    )
    price = scrapy.Field(
        input_processor=MapCompose(remove_tags, filter_price),
        output_processor=TakeFirst(),
    )
```
input and output processors方式的优先级排序如下:
- 在Item Loader 中声明的 field-specific 属性: field_in and field_out (most precedence)
- Item中的字段元数据(input_processor and output_processor key)
- Item Loader 默认处理器: ItemLoader.default_input_processor() and ItemLoader.default_output_processor() (least precedence)

### 内置的处理器
- Identity 最简单的处理器，不进行任何处理，直接返回原来的数据,无参数
```python
>>> from scrapy.loader.processors import Identity
>>> proc = Identity()
>>> proc(['one', 'two', 'three'])
['one', 'two', 'three']
```
- TakeFirst 返回第一个非空(non-null/non-empty)值，常用于单值字段的输出处理器,无参数
```python
>>> from scrapy.loader.processors import TakeFirst
>>> proc = TakeFirst()
>>> proc(['', 'one', 'two', 'three'])
'one'
```
- Join(separator=u' ')返回用分隔符连接后的值,分隔符默认为空格 ,默认为空类似于u''.join
```python
>>> from scrapy.loader.processors import Join
>>> proc = Join()
>>> proc(['one', 'two', 'three'])
u'one two three'
>>> proc = Join('<br>')
>>> proc(['one', 'two', 'three'])
u'one<br>two<br>three'
```
- Compose(*functions, **default_loader_context)给定多个函数组合构造处理器,每个输入值被传递到第一个函数，然后其输出再传递到第二个函数，直到最后一个函数返回整个处理器的输出
默认情况下，当遇到None值的时候停止处理。可以通过传递参数stop_on_none=False改变这种行为,每个函数可以选择接收一个loader_context参数
```python
>>> from scrapy.loader.processors import Compose
>>> proc = Compose(lambda v: v[0], str.upper)
>>> proc(['hello', 'world'])
'HELLO'
```
- MapCompose(*functions, **default_loader_context)

与Compose处理器类似，区别在于各个函数结果在内部传递的方式

    输入值是被迭代的处理的，每一个元素被单独传入第一个函数进行处理。处理的结果被|连接起来(concatenate)形成一个新的迭代器，并被传入第二个函数，以此类推，直到最后一个函数。最后一个函数的输出被连接起来形成处理器的输出。
    每个函数能返回一个值或者一个值列表，也能返回None(会被下一个函数所忽略)
    这个处理器提供了方便的方式来组合多个处理单值的函数。因此它常用与输入处理器，因为用extract()函数提取出来的值是一个unicode strings列表。
```python
>>> def filter_world(x):
...     return None if x == 'world' else x
...
>>> from scrapy.loader.processors import MapCompose
>>> proc = MapCompose(filter_world, unicode.upper)
>>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
[u'HELLO, u'THIS', u'IS', u'SCRAPY']
```
