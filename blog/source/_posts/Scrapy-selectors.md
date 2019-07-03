---
title: Scrapy Selectors
date: 2017-12-02 22:09:31
tags:
  - Selectors
categories:
  - Scrapy
---
#### 使用selectors
##### 构建 Selectors

通过向 Selector 类的构造函数传入 text 或者是 TextResponse 对象来构造 selectors 实例；它会根据传入的类型(input type)自动的去选择最佳的解析规则(XML vs HTML)
```python
>>> from scrapy.selector import Selector
>>> from scrapy.http import HtmlResponse
 
# 通过text来构建
>>> body = '<html><body><span>good</span></body></html>'
>>> Selector(text=body).xpath('//span/text()').extract()
[u'good']
 
# 通过response来构建
>>> response = HtmlResponse(url='http://example.com', body=body)
>>> Selector(response=response).xpath('//span/text()').extract()
[u'good']
 
# 通过.selector来构建
>>> response.selector.xpath('//span/text()').extract()
[u'good']

```
##### 使用selectors
通常通过response.xpath()和response.css()来处理返回的html，xpath通过/text()来返回文本或者属性，css通过::text来返回文本或者属性
	```
	<html>
	 <head>
	  <base href='http://example.com/' />
	  <title>Example website</title>
	 </head>
	 <body>
	  <div id='images'>
	   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
	   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
	   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
	   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
	   <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
	  </div>
	 </body>
	</html>
	```

	```python
	scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html
	#################################
	>>> response.xpath('//title/text()')
	[<Selector xpath='//title/text()' data='Example website'>]
	>>> response.css('title::text')
	[<Selector xpath='descendant-or-self::title/text()' data='Example website'>]
	# 上述结果中，返回的是一个 SelectorList 实例，该实例中包含了一组 selectors；通过调用 SelectorList 的相关接口我们可以获取到每一个 selector 元素的相关内容
	 
	>>> response.css('img').xpath('@src').extract()
	['image1_thumb.jpg', 'image2_thumb.jpg', 'image3_thumb.jpg', 'image4_thumb.jpg', 'image5_thumb.jpg']
	# 通过 extract() 方法便可以从 selector 中提取出所要的文本
	# extract_first() 取第一个元素的值，如果没有返回None，也可以自定义,通过extract_first(default='not-found')
	 
	>>> response.xpath('//base/@href').extract()
	[u'http://example.com/']
	 
	>>> response.css('base::attr(href)').extract()
	[u'http://example.com/']
	 
	>>> response.xpath('//a[contains(@href, "image")]/@href').extract()
	[u'image1.html',
	 u'image2.html',
	 u'image3.html',
	 u'image4.html',
	 u'image5.html']
	 
	>>> response.css('a[href*=image]::attr(href)').extract()
	[u'image1.html',
	 u'image2.html',
	 u'image3.html',
	 u'image4.html',
	 u'image5.html']
	 
	>>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
	[u'image1_thumb.jpg',
	 u'image2_thumb.jpg',
	 u'image3_thumb.jpg',
	 u'image4_thumb.jpg',
	 u'image5_thumb.jpg']
	 
	>>> response.css('a[href*=image] img::attr(src)').extract()
	[u'image1_thumb.jpg',
	 u'image2_thumb.jpg',
	 u'image3_thumb.jpg',
	 u'image4_thumb.jpg',
	 u'image5_thumb.jpg']
	 
	```
##### 嵌套selectors

	通过.xpath()或者.css()方法返回的包含相同类型的selectos 的队列，仍然可以对返回的 selector 执行.xpath()和.css()方法

##### XPath表达式中的参数

	XPath 允许你引用 XPath 表达式中的参数，使用$somevariable
	```python
	>>> # `$val` used in the expression, a `val` argument needs to be passed
	>>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
	u'Name: My image 1 '
	```

#### 常用selectors
##### xpath语法
```
- article   选取所有article元素的所有子节点
- /article 选取根元素article
- article/a 选取所有属于article的子元素的a元素
- //div 选取所有div子元素
- article//div 选取所有属于article元素的后代的div元素，不管它出现在article 之下的任何位置
- //@lang  选取名为lang的所有属性
- //@class='xxx' 选取所有名为class的属性为xxx
- /article/div[1] 选取属于article子元素的第一个div元素
- /article/div[last()] 选取属性article子元素的最后一个div元素
- /article/div[last()-1] 倒数第二个元素
- /article/div[position()<3]选取最前面的两个属于article 元素的子元素的div元素。
- //div[@lang] 选取所有拥有lang属性的div元素
- //div[@lang='eng'] 选取所有lang属性为eng的div元素
- /div/* 选取属于div元素的所有子节点
- //* 选取所有元素
- //div[@*] 选取所有带属性的title元素
- /div/a | //div/p 选取所有div元素的a和p元素
- //span | //ul 选取文档中的span和ul元素
- article/div/p | //span 选取所有属于article元素的div元素的p元素 以及文档中所有的span元素
```
##### css语法
```
- `*` 所有选择器
- #container 选择id=container的元素
- .container 选取class=container的元素
- p     选择所有p元素
- div,p 选择所有div和所有p元素
- li a 选取所有li下的所有a节点
- ul + p 选择u后面的第一个p元素
- div#container > ul 选取id为container的div的第一个ul子元素
- h2 a::text	h2元素下a标签对应的值
- a::attr(href)	a元素中属性为href对应的值
- [target] 选择带有 target 属性所有元素
- [target=_blank] 选择 target="_blank" 的所有元素
- [title~=flower]   选择 title 属性包含单词 "flower" 的所有元素
- [lang|=en]    选择 lang 属性值以 "en" 开头的所有元素
- ul ~ p 选取与ul相邻的所有p元素
- a[title] 选取所有有title属性的a元素
- a[href="http://xxx.com"] 选取所有href属性为xxx.com值的a元素
- a[href*="xxx"] 选取所有href属性包含xxx的a元素
- a[href^="http"] 选取所有href属性值以http开头的a元素
- a[href$=".jpg"] 选取所有href属性值以.jpg结尾的a元素
- input[type=radio]:checked 选择选中的radio的元素
- div:not(#container) 选取所有id非container的div属性
- li:nth-child(3) 选取第三个li元素
- tr:nth-child(2n) 第偶数个tr
```

#### XPATH和CSS用法
| XPATH | CSS |	desc
|--------|--------|-------|
| //div/a| div > a| div的子元素a|
| //div//a| div a| div的后代元素a|
| //div[@id='example']| #example| 获取id=example的元素|
| //div[@class='example']|.example|获取class=example的元素|



















