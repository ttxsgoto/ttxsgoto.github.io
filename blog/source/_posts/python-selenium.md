---
title: Python Selenium模块
date: 2017-12-13 07:39:28
tags:
  - Selenium
  - Scrapy
categories:
  - python
---

#### 说明
Selenium是Thoughtworks公司的一个集成测试的强大工具，Selenium 是 ThoughtWorks 专门为 Web 应用程序编写的一个验收测试工具；使用 Selenium 的最大好处是： Selenium 测试直接在浏览器中运行，就像真实用户所做的一样。在浏览器加载js后，便可以通过xpath来解析网页了。直接pip install selenium 安装完成

#### 实例
```python
#!/usr/bin/env python
# coding: utf-8
import time
import unittest
 
from selenium import webdriver  # selenium.webdriver 模块提供了所有WebDriver的实现,如Firefox, Chrome, IE and Remote
from selenium.webdriver.common.keys import Keys # `Keys`类提供键盘按键的支持
 
class PythonOrgSearch(unittest.TestCase):
 
    def setUp(self):
        self.driver = webdriver.Chrome(executable_path=/usr/bin/chromedriver)     # 创建实例
 
    def test_search(self):
        driver = self.driver
        driver.get("http://www.python.org") # 方法将打开URL中填写的地址，WebDriver 将等待， 直到页面完全加载完毕（其实是等到”onload” 方法执行完毕），然后返回继续执行你的脚本。 值得注意的是，如果你的页面使用了大量的Ajax加载， WebDriver可能不知道什么时候页面已经完全加载
        self.assertIn("Python", driver.title)     # driver.title 表示网页标题
        elem = driver.find_element_by_name("q")
        elem.clear()    # 预先清除input输入框中的任何预填充的文本
        elem.send_keys("pycon") # 输入搜索字
        elem.send_keys(Keys.RETURN) # 发送keys，这个和使用键盘输入keys类似。 特殊的按键可以通过引入`selenium.webdriver.common.keys`的 Keys 类来输入,如enturn,
        time.sleep(10)
        assert "No results found." not in driver.page_source    # driver.page_source 网页html源文件
    
    def test_other(self):
        """常用其他模块说明"""
        browser = self.driver
        browser.get("http://www.python.org")
        elem = browser.find_element_by_css_selector('input#id-search-field')
        elem.clear()
        elem.send_keys('python')
        browser.find_element_by_css_selector('button#submit').click()   # 通过查找对应的元素，然后调用click方法
        
        # 将得到的网页通过scrapy的Selector来解析
        from scrapy.selector import Selector
        selector = Selector(text=browser.page_source)
        submit = selector.css('button#submit::text').extract_first()
        self.assertEqual('Go', submit.strip())
 
    def tearDown(self):
        self.driver.close() # close只会关闭一个标签页; quit关闭整个浏览器
 
 
if __name__ == "__main__":
    unittest.main()
 
```

#### 页面交互
```
<input type="text" name="passwd" id="passwd-id" />
```
查询如下：
```
element = driver.find_element_by_id("passwd-id")
element = driver.find_element_by_name("passwd")
element = driver.find_element_by_xpath("//input[@id='passwd-id']")
```
`注意`：当使用`XPATH`时，你必须注意，如果匹配超过一个元素，只返回第一个元素。 如果上面也没找到，将会抛出 ``NoSuchElementException``异常。

常用方法：
```
element.send_keys("some text")  # 输入内容
element.send_keys(" and some", Keys.ARROW_DOWN)    # 输入方向键
element.clear() # 预先清楚input/textarea中内容
select.deselect_all()   # 取消选择已经选择的元素
driver.find_element_by_id("submit").click() # 提交
driver.switch_to_alert()    # 弹出对话框
driver.forward()    # 浏览历史中的前进
driver.back()   # 浏览历史中的后退
driver.get_cookies()    # 获取cookie
```

#### 查找元素
Selenium提供的方法：
- find_element
- find_element_by_id
- find_element_by_name
- find_element_by_xpath
- find_element_by_link_text
- find_element_by_partial_link_text
- find_element_by_tag_name
- find_element_by_class_name
- find_element_by_css_selector

一次查找多个元素(返回list列表):
- find_elements
- find_elements_by_name
- find_elements_by_xpath
- find_elements_by_link_text
- find_elements_by_partial_link_text
- find_elements_by_tag_name
- find_elements_by_class_name
- find_elements_by_css_selector

---
`查找元素实例`
- find_element_by_id
```html
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
  </form>
 </body>
<html>
 
可以这样查找表单(form)元素
login_form = driver.find_element_by_id('loginForm')
```
- find_element_by_name
```html
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
   <input name="continue" type="button" value="Clear" />
  </form>
</body>
<html>
 
name属性为 username & password 的元素可以像下面这样查找
username = driver.find_element_by_name('username')
password = driver.find_element_by_name('password')
continue = driver.find_element_by_name('continue')
```
- find_element_by_xpath
```html
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
   <input name="continue" type="button" value="Clear" />
  </form>
</body>
<html>
 
可以这样查找表单(form)元素
login_form = driver.find_element_by_xpath("/html/body/form[1]")
login_form = driver.find_element_by_xpath("//form[1]")
login_form = driver.find_element_by_xpath("//form[@id='loginForm']")
 
username = driver.find_element_by_xpath("//form[input/@name='username']")
username = driver.find_element_by_xpath("//form[@id='loginForm']/input[1]")
username = driver.find_element_by_xpath("//input[@name='username']")
 
clear_button = driver.find_element_by_xpath("//input[@name='continue'][@type='button']")
clear_button = driver.find_element_by_xpath("//form[@id='loginForm']/input[4]")
```
- find_element_by_link_text
```html
<html>
 <body>
  <p>Are you sure you want to do this?</p>
  <a href="continue.html">Continue</a>
  <a href="cancel.html">Cancel</a>
</body>
<html>
 
continue.html 超链接可以被这样查找到:
continue_link = driver.find_element_by_link_text('Continue')
continue_link = driver.find_element_by_partial_link_text('Conti')
```
- find_element_by_tag_name
```html
<html>
 <body>
  <h1>Welcome</h1>
  <p>Site content goes here.</p>
</body>
<html>
 
h1 元素可以如下查找
heading1 = driver.find_element_by_tag_name('h1')
```
- find_element_by_class_name
```html
<html>
 <body>
  <p class="content">Site content goes here.</p>
</body>
<html>
 
p 元素可以如下查找
content = driver.find_element_by_class_name('content')
```
- find_element_by_css_selector
```html
<html>
 <body>
  <p class="content">Site content goes here.</p>
</body>
<html>
 
p 元素可以如下查找:
content = driver.find_element_by_css_selector('p.content')
```

#### 等待加载
- 显式等待

    显式等待是你在代码中定义等待一定条件发生后再进一步执行你的代码，最糟糕的案例是使用time.sleep()，它将条件设置为等待一个确切的时间段

- 隐式等待

    如果某些元素不是立即可用的，隐式等待是告诉WebDriver去等待一定的时间后去查找元素


