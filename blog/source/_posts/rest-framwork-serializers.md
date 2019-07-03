---
title: DRF ModelSerializer常用方法
date: 2017-05-14 20:13:22
tags:
  - ModelSerializer
categories:
  - DRF
---

### ModelSerializer属性方法
```python
class AccountSerializer(serializers.ModelSerializer):
    other_name = serializers.CharField(source='name', read_only=True)   # 新添加fields中字段,该serializer对应的model中字段
    other_field = serializers.SerializerMethodField()   # 添加不是该model中的字段
    class Meta:
        model = Account # 指定model
        fields = ('id', 'account_name', 'users', 'created')  # 包括的字段
        fields = '__all__'  # 显示所有字段
        exclude = ('users',)    # 排除不显示的字段,和fields不能同时使用
        depth = 1   # 展示ForeignKey对应的数据，设置展示深度
        read_only_fields = ('account_name',)    # 设置只读字段
    
    def get_other_field(self, obj):# (dept为外键字段)
        return obj.dept.name if obj.dept else ''
```
