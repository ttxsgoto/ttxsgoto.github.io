---
title: Django 分页
date: 2017-03-08 21:57:42
tags:
  - 分页
categories:
  - Django
---
在Django 项目中使用自带的分页模块，记录如下

**相关说明**
```python
from django.core.paginator import Paginator
 
objects = ['abc','bcd','cde','def','efg','fgh']
p = Paginator(objects, 2)#表示传入数据集合，2 表示每页存放的数据个数
p.cout #6 表示数据总量
p.num_pages#3 表示总页数
p.per_page#2 表示每页的数量
p.object_list# ['abc','bcd','cde','def','efg','fgh']  所有数据列表
p.page_range  #[1, 2, 3]   表示页数
p1 = p.page(1)#获取第一页的对象
p1.object_list#['abc', 'bcd'] 该页上的数据
p1.has_next()#True,判断该页是否有下一页
p1.has_previous()#False,判断该页是否有上一页
p1.has_other_pages() #判断该页是否有上或者下页
p1.start_index()# 1 p1开始的索引值
p1.end_index()#2 p1结束的索引值
p1.next_page_number(）# #返回下一页的页码，如果下一页不存在，抛出InvalidPage异常
p1.previous_page_number()#返回前一页的页码，如果上一页不存在，抛出InvalidPag异常
p1.paginator#<django.core.paginator.Paginator object at 0x7f39989b2bd0> 相关paginator的对象
```

**Django中使用分页**
```python
from django.shortcuts import render_to_response
from django.core.paginator import Paginator, EmptyPage, InvalidPage, PageNotAnInteger
from models import *
 
def device_status(request):
    All_page_info = Device_status.objects.all()
    DataCount,page,All_page_info = All_in_one(request,All_page_info)

    return my_render("serverinfo/device_status.html",locals(),request)
 
def my_render(template,data,request):
    return render_to_response(template, data, context_instance=RequestContext(request))
 
def getpage_id(request):
    """
    传递前端page过来的值，如果没有设置为1
    """
    page = request.GET.get("page","")
    if page:
        page = request.GET.get("page","")
    else:
        page = 1
    return page
 
def getpages(Data,page):
    """
    处理分页函数
    """
    paginator = Paginator(list(Data),2)
    try:
        Data = paginator.page(page)
    except PageNotAnInteger:
        Data = paginator.page(1)
    except EmptyPage:
        Data = paginator.page(paginator.num_pages)
    return Data,page
 
def All_in_one(request,All_page_info):
    DataCount = len(All_page_info)
    page = getpage_id(request)
    All_page_info,page=getpages(All_page_info,page)
    return DataCount,page,All_page_info
```
**templates中**

```
备注：paginator.html  //将以下模板导入需要添加分页功能的页面即可（{% include 'paginator.html' %}）
=========================
<span class="step-links">
   {% if All_page_info.has_previous %} <!-- 是否有前一页 -->
       <a  href="?page={{ All_page_info.previous_page_number }}">上一页</a> <!-- 前一页的页码 -->
         </script>
   {% endif %}
   <span class="current">
       第 {{ All_page_info.number }}页  总 {{ All_page_info.paginator.num_pages }} 页  <!-- 总页数 -->
   </span>
   {% if All_page_info.has_next %}
       <a  href="?page={{ All_page_info.next_page_number }}">下一页</a> <!-- 后一页的页码 -->
   {% endif %}
</span>
<span>
     &nbsp;&nbsp;共{{DataCount}}条记录
</span>
```
**效果展示**
![](https://ttxsgoto.github.io/img/django/page.png)





