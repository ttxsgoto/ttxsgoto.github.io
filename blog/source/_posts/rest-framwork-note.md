---
title: DRF Rest_framwork笔记
date: 2017-05-14 19:36:53
tags:
  - DRF
categories:
  - DRF
---

### 实现restful操作步骤
0.setting.py 添加设置
```python
INSTALLED_APPS = (
	'rest_framework',
        'app',
)

```
1.serializer序列化models
```python
from rest_framework import serializers
from django.contrib.auth.models import User
 
#使用模型序列化ModelSerializer
class UserSerializer(serializers.ModelSerializer):
    """
    用户管理
    """
    class Meta:
        model = User
        fields = ('id', 'username', 'password')
 
    def create(self, validated_data):   # 方法重写
        user = User.objects.create(**validated_data)
        return user
 
#简单的默认create()和update()方法的实现
```
2.viewSet中：
```python
queryset = user.objects.all()       # 取数据all
serializer_class = UserSerializer   # 将序列化的数据给
 
# 或者返回数据列表
departments = Department.objects.all()
response.data.update({'departments': DepartmentSerializer(list(departments), many=True).data})
```
3.注册router
```python
from rest_framework.routers import DefaultRouter
router =  routers.DefaultRouter()
router.register(r'users', UserViewSet)
```
### 实例化(Serializer)类和模型实例化(ModelSerializer)类
serializers.py  #序列化类的快捷方式，同时包括create,update方法
```python
from rest_framework import serializers	    # 序列化
from django.contrib.auth.models import User	# 导入model
 
class UserSerializer(serializers.ModelSerializer):
    """
    用户管理
    """
 
    class Meta:
        model = User  # 定义models
        fields = ('id', 'username', 'first_name', 'email', 'password')	# 字段
 
```

### 请求对象
```
request.data #可以处理任何数据，对post，put，patch等方法也起作用
```

### 响应对象
```
from rest_framework.response import Response
return Response()	
# 根据客户端的请求来渲染成指定的内容类型,用于未渲染内容和内容协商来决定正确的内容类型并把它返回给客户端的模板响应(TemplateResponse).
```

### 状态码
```python
# REST框架为每个状态码提供了明确的标识符，如HTTP_400_BAD_REQUEST等
 
from rest_framework import status
 
def create(self,request):
	return Response(data, status=status.HTTP_201_CREATED)
```

### 装饰API视图

- @app_view 用在基于视图的方法上

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
 
@api_view(['GET', 'PUT', 'DELETE', 'POST'])
def index(request):
    print request.method
    print request.DATA
    return Response([{'asset': '1','status': 'ok'}])
 
@api_view(['GET', 'PUT', 'DELETE', 'POST'])
def list_api(request):
    if request.method == 'GET':
    	pass
        return Respose(serializer.data)
    elif request.method == 'POST':
    	pass
        return Response(data,status=status.HTTP_206_PARTIAL_CONTENT)
 
#urls.py中设置
from rest_framework.urlpatterns import format_suffix_patterns
 
urlpatterns = format_suffix_patterns(urlpatterns)
```

- APIView 用在基于视图的类

```python
# 基于视图的类
views.py中
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
 
class UserView(APIView):
	def get(self,request,format=None):
    	objectall = User.objects.all()
        serializer = UserSerializer(objectall)
        return Response(serializer.data)
 
    def post(self, request, formate=None):
    	pass
 
# urls.py中
from rest_framework.urlpatterns import format_suffix_patterns
 
    url(r'^app/user/$', UserView.as_view()),
    url(r'^app/user/(?P<pk>[0-9]+)$', UserView.as_view()),
 
urlpatterns = format_suffix_patterns(urlpatterns)
```

- 使用基于视图的一般类(generic class)

```python
# views.py
    queryset = info.objects.all()    #models对象
    serializer_class = UserSerializer	#序列化的类
```

### 授权(Authentication)与权限(Permissions)
- 数据总是和创建者联系在一起
- 只有授权用户才能创建对应的数据
- 只有对应的数据的创建者才能更新或删除它
- 没有授权的请求应该只有只读权限

```python
# views.py中
from rest_framework import permissons
 
urls.py中
urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]
################################
# 在浏览器API中添加登录
urls.py
from django.conf.urls import url, include
 
urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]	# 为api添加一个包括登录和退出视图的url样式
 
# api-auth部分可以是任何你想要的url，唯一的限制为include中的链接必须使用‘rest_framework’名字空间，在Django1.9+ rest框架会设置名字空间，所以必须写
```

### URL模式命名
```python
url(r'^app/user/$', UserView.as_view(), name='user_add'), # name指定url名称

```

### 添加分页
```python
# setting.py中
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'base.serializers.PlugPageNumberPagination',
    'MAX_PAGE_SIZE': 50,
    'PAGE_SIZE':     15  # default page size
}
```

### 视图集(ViewSets)和路由(Routers)
视图集让开发者把精力集中在构建状态和交互的api模型，而且他可以基于一般规范自动构建url
一个viewset类只绑定一个方法集合，当它初始化一个视图集合时，一般使用为处理复杂的url定义的Router类
```python
# views.py
from rest_framework import viewsets
 
class UserviewSet(viewsets.ModelViewSet):
	queryset = User.objects.all()
	serializer_class = UserSerializer
    def list(self,request):
    	pass
        return data
 
```
### 明确绑定视图集到URL
定义URLConf时，处理方法只绑定了动作，我们必须从我们的视图集(ViewSets)创建一个视图集合，在urls.py文件中，我们将ViewSet类绑定到具体视图的集合

```python
# urls.py中
from rest_framework.urlpatterns import format_suffix_patterns
 
user_list = UserView.as_view({
    'get':'list',
    'post':'creat',
})
url(r'^user/$, user_list)),
 
# 或者：
list_create = {
    'get': 'list',
    'post': 'create',
}
 url(r'^user/$', UserView.as_view(list_create)),
```

### 使用路由
使用Router类可以自动将资源和视图(views),链接(urls)联系起来，我们只需要用一个路由注册合适的视图集合
```python
# urls.py中
from rest_framework.routers import DefaultRouter
 
router =  routers.DefaultRouter()
router.register(r'users',views.UserViewSet)
router.register(r'groups',views.GroupsViewSet)
 
urlpatterns += [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]
```
### 参考链接
http://www.django-rest-framework.org/
https://github.com/tomchristie/rest-framework-tutorial
http://www.cnblogs.com/loveis715/p/4669091.html












