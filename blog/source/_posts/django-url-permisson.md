---
title: Django 基于URL权限管理模块
date: 2018-01-17 22:38:28
tags:
  - permisson
categories:
  - Django
---

#### 背景
Django自带的权限系统为model层面的，也就是说给某个用户或者组对应的model权限，拥有了操作该model的权限，不能更精细化的控制权限，如对象级别的权限

现在引用了基于url的权限系统，可以控制用户访问一个接口的不同方法 ，可以很方便的控制权限；如：用户对于某个接口可以做到只拥有查看的权限而没有修改的权限，因为对于相同的url请求的方法不同

#### 功能说明
1.类似于django自身权限模块，拥有用户和用户组两类权限
2.通过swagger 获取项目URL 列表用于分配权限
3.通过中间件来拦截判断用户对请求的url是否有对应的权限

#### 主要代码说明
##### models.py
```python
#!/usr/bin/env python
# coding: utf-8
 
from __future__ import unicode_literals
from django.db import models
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import Permission, User
from .managers import UserprofileManager
 
 
class Userprofile(models.Model):
    """用户相关信息"""
 
    user = models.OneToOneField(settings.AUTH_USER_MODEL, primary_key=True)
    sex = models.IntegerField(choices=(
        (0, '女性'),
        (1, '男性'),
        (2, '未填写')
    ), default=2, help_text='sex')
    groups = models.ManyToManyField(
        'Group',
        verbose_name='user groups',
        blank=True,
        through='UserGroup',
        through_fields=('user', 'group')
    )
    user_permissions = models.ManyToManyField(
        'URLSource',
        verbose_name='user permissions',
        blank=True,
        through='UserURL',
        through_fields=('user', 'url')
    )
    desc = models.CharField(u'描述', null=True, blank=True, max_length=100, default='')
    create_time = models.DateTimeField(auto_now_add=True)
 
    objects = UserprofileManager()
 
    class Meta:
        db_table = 'user_profile'
        verbose_name = u'用户信息'
        verbose_name_plural = verbose_name
        ordering = ['-create_time']
 
    def __str__(self):
        return '{}'.format(self.user.username)
 
    @receiver(post_save, sender=User)
    def create_user_profile(sender, instance=None, created=False, **kwargs):
        if created:
            Userprofile.objects.get_or_create(user=instance, defaults={'desc': instance.username})
 
    def add_permissions(self, permissions):
        """用户添加权限"""
        history_permission = self.userurl_set.all()  # 现在已有的权限
        if history_permission:
            # 如果有就不添加,没有再添加,多余的删除
            for _permiss in history_permission:
                if _permiss.url in permissions:
                    permissions.remove(_permiss.url)
                else:
                    _permiss.delete()
            permisson_list = [UserURL(user=self, url=permisson) for permisson in permissions]
        else:
            permisson_list = [UserURL(user=self, url=permisson) for permisson in permissions]
        self.userurl_set.bulk_create(permisson_list)
        return self
 
    def add_groups(self, group):
        """
        用户添加到组(目前只支持添加到单个组)
        :param group: 权限组实例
        :return:
        """
        user_group = self.usergroup_set.all()
        if user_group:
            user_group.delete()
        self.usergroup_set.create(group=group)
        return self
 
    def get_group_permissions(self):
        """获取用户所在组的权限"""
        return self.usergroup_set.all()
 
    def get_url_permissions(self):
        """获取用户单独关联的权限"""
        return self.userurl_set.all()
 
    def get_all_permissions(self):
        """该用户的所有权限,包括所在组权限和自己单独关联权限"""
        permissions = set()
        # group 权限
        group_permission = self.get_group_permissions()
        for permission in group_permission:
            URL = GroupURL.objects.filter(group=permission.group)
            for _permission in URL:
                permissions.add(_permission.url)
        # user 权限
        url_permission = self.get_url_permissions()
        for url in url_permission:
            permissions.add(url.url)
        return permissions
 
 
class URLSource(models.Model):
    """URL资源信息"""
 
    url = models.CharField(u'url', max_length=128)
    action = models.CharField(u'请求方法', max_length=16)
    description = models.CharField(u'描述信息', max_length=256, blank=True, null=True, default='')
    parameters = models.CharField(u'参数, 逗号分隔', max_length=128, default='', blank=True, null=True)
 
    class Meta:
        verbose_name = u'URL信息'
        verbose_name_plural = verbose_name
        db_table = 'url_source'
        unique_together = (('url', 'action'),)
        ordering = ['id']
 
    def __str__(self):
        return '{}-{}-{}'.format(self.id, self.description, self.action)
 
 
class Group(models.Model):
    """用户组信息"""
 
    name = models.CharField(max_length=64, verbose_name='组名', unique=True, help_text='组名')
    code = models.CharField(max_length=64, verbose_name='用户组中文名', default='', help_text='组code')
    permissions = models.ManyToManyField(
        URLSource,
        verbose_name='permissions',
        blank=True,
        through='GroupURL',
        through_fields=('group', 'url')
    )
 
    class Meta:
        db_table = 'group'
        verbose_name = u'用户权限组'
        verbose_name_plural = verbose_name
 
    def __str__(self):
        return self.name
 
    def add_permissions(self, permissions):
        """
        给组添加权限
        :param permissons: 权限列表,filter查询列表
        :return:
        """
        history_permission = self.groupurl_set.all()  # 现在已有的权限
        if history_permission:
            # 如果有就不添加,没有再添加,多余的删除
            for _permiss in history_permission:
                if _permiss.url in permissions:
                    permissions.remove(_permiss.url)
                else:
                    _permiss.delete()
            permisson_list = [GroupURL(group=self, url=permisson) for permisson in permissions]
        else:
            permisson_list = [GroupURL(group=self, url=permisson) for permisson in permissions]
        self.groupurl_set.bulk_create(permisson_list)
        return self
 
    def get_group_permissions(self):
        """获取该组对应的权限列表"""
        return self.groupurl_set.all()
 
 
class UserGroup(models.Model):
    """用户和组关联"""
 
    user = models.ForeignKey(Userprofile, verbose_name=u'用户')
    group = models.ForeignKey(Group, verbose_name=u'组')
    desc = models.CharField(u'其他信息', max_length=100, blank=True, null=True, default='')
 
    class Meta:
        db_table = 'user_groups'
        verbose_name = u'用户和组关系'
        verbose_name_plural = verbose_name
        unique_together = (('user', 'group'),)
 
    def __str__(self):
        return '{}-{}'.format(self.user, self.group)
 
 
class UserURL(models.Model):
    """用户和URL关联"""
 
    user = models.ForeignKey(Userprofile, verbose_name=u'用户')
    url = models.ForeignKey(URLSource, verbose_name=u'URL')
    desc = models.CharField(u'其他信息', max_length=100, blank=True, null=True, default='')
 
    class Meta:
        db_table = 'user_urls'
        verbose_name = u'用户权限关系'
        verbose_name_plural = verbose_name
        unique_together = (('user', 'url'),)
 
    def __str__(self):
        return '{}-{}'.format(self.user, self.url)
 
 
class GroupURL(models.Model):
    """组和URL关系"""
 
    group = models.ForeignKey(Group, verbose_name=u'组')
    url = models.ForeignKey(URLSource, verbose_name=u'URL')
    desc = models.CharField(u'其他信息', max_length=100, blank=True, null=True, default='')
 
    class Meta:
        db_table = 'group_urls'
        verbose_name = u'用户权限组关系'
        verbose_name_plural = verbose_name
        unique_together = (('group', 'url'),)
 
    def __str__(self):
        return '{}-{}'.format(self.group, self.url)
```
##### middleware.py
```python
#!/usr/bin/env python
# coding: utf-8
 
from __future__ import unicode_literals
import logging
import json
from collections import Counter
from django.http import HttpResponseForbidden
from django.utils.deprecation import MiddlewareMixin
from .models import Userprofile
from django.contrib.auth import get_user_model
from .settings import url_permisson_settings
 
logs = logging.getLogger('django')
 
User = get_user_model()
 
 
class URLPermissionMiddleWare(MiddlewareMixin):
    def process_request(self, request):
        response = {
            "status_code": 403,
            "message": u"无权限操作,请联系管理员."
        }
        path = request.path.split('/')
        if path[1] in url_permisson_settings.ALL_ALLOW_URL:
            return None
        if not isinstance(request.user, User):
            return None
        if request.user.is_superuser:
            return None
        try:
            profile = Userprofile.objects.get(user=request.user)
        except Userprofile.DoesNotExist:
            return HttpResponseForbidden(json.dumps(response), content_type='application/json')
        method = request.method.lower()
        path = request.path.strip()
        if method == 'get':
            parameter = []
            for _parameter in request.GET.items():
                parameter.append(_parameter[0])
        else:
            try:
                parameter_dict = json.loads(request.body)
            except Exception:
                parameter_dict = {}
            parameter = parameter_dict.keys()
        all_permission = profile.get_all_permissions()
        result = None
        for _url in all_permission:
            url = _url.url
            action = _url.action
            parameters = _url.parameters.split(',')
            if method == action and path == url and (len(Counter(parameter) - Counter(parameters)) == 0):
                result = True
        if not result:
            return HttpResponseForbidden(json.dumps(response), content_type='application/json')
        return None
 
```
#### 接口说明
![](https://ttxsgoto.github.io/img/django/permission.png)


#### Git地址
https://github.com/ttxsgoto/url_permission



