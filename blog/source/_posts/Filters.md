---
title: DRF Filters
date: 2017-08-11 20:10:55
tags:
  - Filters
categories:
  - DRF
---
DRF通过 url参数 来对数据进行一些排序或过滤的操作,rest-framwork提供了filters来满足这一需求

#### 全局filter
```
pip install django-filter
```
在 settings 里指定应用到全局的 filter
```
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ('rest_framework.filters.DjangoFilterBackend',)
}
```
#### filter定义字段
filters.py
```python
from django_filters import FilterSet
 
class UserRecordFilter(FilterSet):
    class Meta:
        model = UserRecord
        fields = ['source', 'source_id', 'url',]
        fields = {
        'from_channel':['gt', 'lt', 'in', 'exact', 'range', 'isnull', 'icontains']
            # 大于, 小于，多个，等于, 范围, 是否为空bool,模糊查询
        }
        exclude = ['from_channel', 'created_time'] # 排除字段
        together = ['from_channel', 'resume_id'] # 字段并集，同时满足条件
```
views.py
```python
class UserRecordViewSet(ModelViewSet):
    serializer_class = UserRecordSerializer
    queryset = UserRecord.objects.all()
    filter_class = UserRecordFilter
    filter_backends = (DjangoFilterBackend,)

# 请求url
# http://127.0.0.1:8888/api/v2/user_records?source_id=3&url=url
```

#### viewset 的 filter
为 viewset 分别指定 filter，方法就是在定义 viewset 的时候定义一个名为 filter_backend 的类变量：
```python
class UserListView(generics.ListAPIView):
    queryset = User.objects.all()
    serializer = UserSerializer
    filter_backends = (filters.DjangoFilterBackend,)
```
#### 默认的 filter
- SearchFilter
```python
filter_backends = (filters.SearchFilter,)
search_fields = ('username', 'email')  # 指定搜索的域
 
# 请求url
# http://127.0.0.1:8888/api/v2/user_records?search=username
```
- OrderingFilter
```python
filter_backends = (filters.OrderingFilter,)
ordering_fields = ('created_time')
 
# 请求url
# http://127.0.0.1:8888/api/v2/user_records?ordering=-created_time
```
#### 自定义 filter
自定义filter，只需要定义 filter_queryset(self, request, queryset, view) 方法，并返回一个queryset即可
```python
class NodenameFilter(filters.BaseFilterBackend):

    """
    根据 nodename 来筛选[nodename]
    """
 
    def filter_queryset(self, request, queryset, view):
        nodename = request.QUERY_PARAMS.get('nodename')
        if nodename:
            return queryset.filter(nodename=nodename)
        else:
            return queryset
```

#### 参考链接
https://django-filter.readthedocs.io/en/develop/index.html#
