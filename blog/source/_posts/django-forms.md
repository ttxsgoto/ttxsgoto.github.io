---
title: Django Forms表单
date: 2017-07-15 21:17:58
tags:
  - Forms
categories:
  - Django
---
### form表单验证顺序步骤：
1. to_python()将值转换为正确的pthon数据类型，如果不能转换，抛出ValidationError
2. 字段的validate()方法处理字段特殊定义的验证
3. 字段的run_validators() 方法运行字段的所有Validator，并将所有的错误信息聚合成一个单一的ValidationError
4. Field子类的clean() 方法，负责以正确的顺序运行to_python、validate 和 run_validators 并传播它们的错误。如果任何时刻、任何方法引发ValidationError，验证将停止并引发这个错误；这个方法返回验证后的数据，这个数据在后面将插入到表单的 cleaned_data 字典中
5. 表单子类中的clean_<fieldname>() 方法，这个方法完成于特定属性相关的验证，自定义字段验证，数据在clean_data字典中，该方法返回从cleaned_data 中获取的值
6. 表单子类的clean() 方法，这个方法可以实现需要同时访问表单多个字段的验证

对于表单中的每个字段（按它们在表单定义中出现的顺序），先运行Field.clean() ，然后运行clean_<fieldname>()。每个字段的这两个方法都执行完之后，最后运行Form.clean() 方法，无论前面的方法是否抛出过异常。

### 实例：表单字段的默认验证
```python
from django import forms
from django.core.validators import validate_email
 
class MultiEmailField(forms.Field):
    def to_python(self, value):
        "Normalize data to a list of strings."
 
        # Return an empty list if no input was given.
        if not value:
            return []
        return value.split(',')
 
    def validate(self, value):
        "Check if value consists only of valid emails."
 
        # Use the parent's handling of required fields, etc.
        super(MultiEmailField, self).validate(value)
 
        for email in value:
            validate_email(email)
 
########### 创建form类 #########
class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField()
    sender = forms.EmailField()
    recipients = MultiEmailField()
    cc_myself = forms.BooleanField(required=False)
 
# 当调用表单的is_valid()方法时，MultiEmailField.clean()方法将作为验证过程的一部分运行，即将调用to_python()和validate()方法
```

### Validator
验证器是一个可调用的对象，它接受一个值，并在不符合一些规则时抛出
```python
def validate_begin(value):
    if not value.startswith(u'ABC'):
        raise ValidationError('名称不是以ABC开头', code='error_begin')
 
### 调用
other_field = forms.CharField(max_length=100, validators=[validate_begin])
```
### forms常用方法
- f.is_bound 属性说明表单是否具有绑定的数据
- f.is_valid() 验证提交的表单字段是否正确（is_valid() 返回True）
- form.cleaned_data 字典存为验证后的表单数据
- f.errors 获取错误信息的一个字典
- f.errors.as_data() 返回字典，映射到原始的ValidationError实例
- f.errors.as_json() 返回json序列化后的错误
- f.has_changed() 检查表单的数据是否从初始数据发生改变
- f.fields 从表单实例的fields属性中访问字段

### save()方法
根据表单绑定的数据创建并保存数据库对象
- instance 关键字接收一个已经存在的实例对象，如果有，则save()将更新这个实例，没有save()将创建一个新的实例
- commit save() 关键字参数，其值为True 或False。如果save() 时commit=False，那么它将返回一个还没有保存到数据库的对象,操作后调用save方法保存
- commit=False 时因不能立即生成实例，多对多数据时需要对froms实例调用save_m2m()方法对多对多数据进行保存，如：channel.save() ,_channel = ChannelForm(req.data).save_m2m()

### ModelForm中Meta属性
```python
class ChannelCustomForm(ModelForm):
 
    class Meta:
        model = ChannelCustom
        fields = ['name', 'logo', 'is_valid', 'note']   # 需要编辑的字段
        fields = '__all__'  # 所有字段
        exclude = ['title'] # 排除某些字段
        labels= None        # 用于自定义标签的名字，默认情况下是数据库中表的列名
        help_texts = None     # 帮助提示信息
        widgets = None        # 自定义插件
        error_messages = None # 自定义错误信息
        field_classes = None  # 自定义字段类（也可以自定义字段)
        Localized_fields = (‘birth_date’,) # 本地化，如：根据不同时区显示数据
```
### 实例(说明功能)
- models.py
```python
class ChannelCustom(BaseModel):
 
    organ = models.ForeignKey('organs.Organ', null=True)
    name = models.CharField(u'名称', max_length=40, null=True, blank=True, default='')
    logo = models.CharField(u'渠道logo', max_length=80, null=True, blank=True, default='')
    is_valid = models.BooleanField(u'有效', default=True)
    note = models.CharField(u'备注', max_length=512, null=True, blank=True, default='')
```
- forms.py
```python
from django.forms import ModelForm, Textarea
from django.forms import fields as MFfields
from django import forms
from django.core.exceptions import ValidationError
# from django.utils.translation import ugettext as _
from django.utils.translation import ugettext_lazy as _
 
def validate_begin(value):
    if not value.startswith(u'ABC'):
        raise ValidationError('名称不是以ABC开头', code='error_begin')
 
 
class ChannelCustomForm(ModelForm):
    name = MFfields.CharField(max_length=100)   # 重新定义字段属性,可以为已有的字段,也可以是没有的字段
    other_field = forms.CharField(max_length=100, validators=[validate_begin])
 
 
    def __init__(self, *args, **kwargs):                        # 通过继承来解决对应字段的属性
        super(ChannelCustomForm, self).__init__(*args, **kwargs)
        self.fields['name'].validators.append(validate_begin)   # 添加验证方法
        self.fields['name'].required = True                     # 添加对应字段的属性
        self.fields['note'].required = True
 
    class Meta:
        model = ChannelCustom
        fields = ['name', 'logo', 'is_valid', 'note', 'other_field']
        # exclude = ['logo']
        labels = {          # 用于自定义标签的名字，默认情况下是数据库中表的列名
            "name": "渠道名称",
            "logo": "渠道logo",
            "is_valid": "是否有效",
            "note": "备注"
        }
 
        help_texts = {      # 帮助提示信息
            "name": "请输入渠道名",
            "note": "请输入备注",
        }
 
        error_messages = {  # 自定义错误描述
            # '__all__': {
            #
            # },
            'organ': {
                'max_length': ("企业字段不能为空."),
            },
            'note': {
                'required': "note字段不能为空.",  # 这里的key是特定的字符
                'invalid': 'http格式错误',
            },
        }
        widgets = {     # 自定义 widget，添加对应字段的属性
            'name': Textarea(attrs={'cols': 80, 'rows': 20}),
        }
 
        field_classes = {       # 字段类型设置，可以强制修改成其他类型
            'note': MFfields.URLField
        }
 
        Localized_fields = ('is_valid', )    #本地化，如：根据不同时区显示数据
 
    def clean_name(self):
        """
        定义字段检查方法
        clean()和clean_<field>&()的最后必须返回验证完毕或修改后的值
        :return:
        """
        name = self.cleaned_data['name']
        if not name:
            raise forms.ValidationError('名称不能为空')
        return name
 
    def clean_logo(self):
        logo = self.cleaned_data['logo']
        if not logo:
            raise forms.ValidationError('logo不能为空')
        return logo
 
    def clean(self):
        """
        - 如果你需要覆盖clean() 方法并维持这个验证行为，你必须调用父类的clean()方法
        - 在表单数据提交的时候,所有的数据都会经过clean()函数
        - 用于验证字段间有关联的数据验证
        """
        cleaned_data = super(ChannelCustomForm, self).clean()
 
        print cleaned_data
        logo = cleaned_data.get('logo', '')
        name = cleaned_data.get('name', '')
        if logo != name:
            msg = u'两者不一致相等 %(name)s <--> %(logo)s'
            error_instance = ValidationError(_(msg), code='invalid', params={'name': name, 'logo': logo})
            self.add_error('logo',error=error_instance)     # 添加错误信息
            self.non_field_errors()
            # self.errors['logo'] = self.error_class([msg])
            # raise forms.ValidationError(self.errors['logo'])
            # raise forms.ValidationError('两者不一致相等')
        return cleaned_data
```
- views.py
```python
from .forms import ChannelCustomForm
from django.forms.models import modelformset_factory, modelform_factory
    
def post(self, req):
    
    organ = req.user.get_profile().organ
    data = req.data
    # _channel = ChannelCustom.objects.get(pk=8)
    # form = ChannelCustomForm(data)#, instance=_channel)
    # modelform_factory() 来代替使用类定义来从模型直接创建表单,用于不在很多自定义的情况下
    ChannelCustomFormfactory = modelform_factory(ChannelCustom,
                                                  fields=('name', 'logo', 'is_valid', 'note')
                                                )
 
    # modelformset_factory() 模型表单集
    ChannelCustomSet = modelformset_factory(ChannelCustom,
                                            fields=('name', 'logo', 'is_valid', 'note')
                                            )
 
    form = ChannelCustomForm(data)#, instance=_channel)
    # form = ChannelCustomFormfactory(data=data)
    # form = ChannelCustomSet(data=data)
    if form.is_valid():
        print form.cleaned_data
        print form.errors
        channel = form.save()
        channel.organ= organ
        channel.save()
 
        return resp.serialize_response(channel, results_name='channel')
    error =form.errors.as_json()
    error_msg = json.loads(error)
    return resp.failed(error_msg)
```
### 参考链接
http://python.usyiyi.cn/translate/django_182/topics/forms/modelforms.html




