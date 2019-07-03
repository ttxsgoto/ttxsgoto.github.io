---
title: Python Re模块
date: 2017-03-30 21:18:15
tags:
  - Re
categories:
  - python
---
#### 说明
正则表达式用于字符的筛选和匹配，相关语法规则请参考其他文档，这里主要学习相关函数功能；

#### 匹配使用函数
- re.match(pattern, string[, flags]):从第一个字符开始匹配,匹配成功就返回,不关心后面的内容;
- re.search(pattern, string[, flags]):扫描整个string查找匹配,匹配成功就返回,不关心后面的内容;
- re.split(pattern, string[, maxsplit]):按能够匹配的子串将string分割后返回列表;
- re.findall(pattern, string[, flags]):搜索string，以列表形式返回全部能匹配的子串;
- re.finditer(pattern, string[, flags]):搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器;
- re.sub(pattern, repl, string[, count]):使用repl替换string中每一个匹配的子串后返回替换后的字符串;
- re.subn(pattern, repl, string[, count]):返回 (sub(repl, string[, count]), 替换次数);

#### 实例
```python
#!/usr/bin/env python
#coding: utf_8
 
'''
    Describe:
    re.compile(string[,flag])   #返回pattern对象
    pattern = re.compile(r'hello')
    flag参数是匹配模式，取值可以使用按位或运算符’|’表示同时生效，比如re.I | re.M
    re.I(全拼：IGNORECASE): 忽略大小写（括号内是完整写法，下同）
    re.M(全拼：MULTILINE): 多行模式，改变'^'和'$'的行为（参见上图）
    re.S(全拼：DOTALL): 点任意匹配模式，改变'.'的行为
    re.L(全拼：LOCALE): 使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
    re.U(全拼：UNICODE): 使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
    re.X(全拼：VERBOSE): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。
'''
 
import re
 
def match():
    """
    re.match(pattern, string[, flags])
    这个方法将会从string（我们要匹配的字符串）的开头开始，
    尝试匹配pattern，一直向后匹配，如果遇到无法匹配的字符，
    立即返回None，如果匹配未结束已经到达string的末尾，也会返回None。
    两个结果均表示匹配失败，否则匹配pattern成功，同时匹配终止,不再对string向后匹配。
    :return:
    从第一个字符开始匹配,一旦匹配成功就返回,不关心后面的内容
    ============match属性和方法==============
    # 属性
    1.string: 匹配时使用的文本。
    2.re: 匹配时使用的Pattern对象。
    3.pos: 文本中正则表达式开始搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。
    4.endpos: 文本中正则表达式结束搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。
    5.lastindex: 最后一个被捕获的分组在文本中的索引。如果没有被捕获的分组，将为None。
    6.lastgroup: 最后一个被捕获的分组的别名。如果这个分组没有别名或者没有被捕获的分组，将为None。
 
    # 方法
    1.group([group1, …]):获得一个或多个分组截获的字符串；指定多个参数时将以元组形式返回。group1可以使用编号也可以使用别名；编号0代表整个匹配的子串；不填写参数时，返回group(0)；没有截获字符串的组返回None；截获了多次的组返回最后一次截获的子串。
    2.groups([default]):以元组形式返回全部分组截获的字符串。相当于调用group(1,2,…last)。default表示没有截获字符串的组以这个值替代，默认为None。
    3.groupdict([default]):返回以有别名的组的别名为键、以该组截获的子串为值的字典，没有别名的组不包含在内。default含义同上。
    4.start([group]):返回指定的组截获的子串在string中的起始索引（子串第一个字符的索引）。group默认值为0。
    5.end([group]):返回指定的组截获的子串在string中的结束索引（子串最后一个字符的索引+1）。group默认值为0。
    6.span([group]):返回(start(group), end(group))。
    7.expand(template):将匹配到的分组代入template中然后返回。template中可以使用\id或\g、\g引用分组，但不能使用编号0。\id与\g是等价的；但\10将被认为是第10个分组，如果你想表达\1之后是字符’0’，只能使用\g0。
    """
 
    pattern = re.compile(r'hello')  #把正则表达式编译成一个正则表达式对象,再使用
    result1 = re.match(pattern, 'hello')
    result2 = re.match(pattern, 'hello0 xxx')
    result3 = re.match(pattern, 'heloo xxx')
    result4 = re.match(pattern, 'hello xxx')
 
    print result1,result2,result3,result4   #re对象，<_sre.SRE_Match object at 0x1053d55e0> <_sre.SRE_Match object at 0x1053ba098> None <_sre.SRE_Match object at 0x1053ba100>
    if result1:					#hello
        print result1.group()
    else:
        print 'result1 faild'
 
    if result2:					#hello
        print result2.group()
    else:
        print 'result2 faild'
 
    if result3:					#result3 faild
        print result3.group()
    else:
        print 'result3 faild'
 
    if result4:					#hello
        print result4.group()
    else:
        print 'result4 faild'
 
    print '======================================='
    m = re.match(r'(\w+) (\w+)(?P<sign>.*)', 'hello world!!!')
    print 'm.string:', m.string                             #m.string: hello world!!!
    print 'm.re:', m.re                                     #m.re: <_sre.SRE_Pattern object at 0x103d62690>
    print 'm.pos', m.pos                                    #m.pos 0
    print 'm.endpos', m.endpos                              #m.endpos 14
    print 'm.lastindex:', m.lastindex                       #m.lastindex: 3
    print 'm.lastgroup:', m.lastgroup                       #m.lastgroup: sign
    print 'm.group():', m.group()                           #m.group(): hello world!!!
    print 'm.group(1,2):', m.group(1,2)                     #m.group(1,2): ('hello', 'world')
    print 'm.groups():', m.groups()                         #m.groups(): ('hello', 'world', '!!!')
    print 'm.groupdict()', m.groupdict()                    #m.groupdict() {'sign': '!!!'}
    print 'm.start(2):', m.start(2)                         #m.start(2): 6
    print 'm.end(2):', m.end(2)                             #m.end(2): 11
    print 'm.span(2):', m.span(2)                           #m.span(2): (6, 11)
    print r"m.expand(r'\g \g\g'):", m.expand(r'\2 \1\3')    #m.expand(r'\g \g\g'): world hello!!!
 
def search():
    """
    re.search(pattern, string[, flags])
    search()会扫描整个string查找匹配;
    match（）只有在0位置匹配成功的话才有返回，
    如果不是开始位置匹配成功的话，match()就返回None。
    search方法的返回对象同样match()返回对象的方法和属性
    :return:
    """
    pattern = re.compile(r'world')
    match = re.search(pattern, 'hello world!!!! world')
    if match:
        # 使用Match获得分组信息
        print match.group()     #world
 
def split():
    """
    re.split(pattern, string[, maxsplit])
    按照能够匹配的子串将string分割后返回列表。
    maxsplit用于指定最大分割次数，不指定将全部分割
    :return:
    """
    pattern = re.compile(r'\d+')
    print re.split(pattern, 'xxxx1wsd2dsafds4dafd8',maxsplit=10)    #['xxxx', 'wsd', 'dsafds', 'dafd', '']
 
def findall():
    """
    re.findall(pattern, string[, flags])
    搜索string，以列表形式返回全部能匹配的子串
    :return:
    """
    pattern = re.compile(r'\d+')
    print re.findall(pattern, 'xxxx1wsd2dsafds4dafd8',)     #['1', '2', '4', '8']
 
def finditer():
    """
    re.finditer(pattern, string[, flags])
    搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器
    :return:
    """
    pattern = re.compile(r'\d+')
    for m in re.finditer(pattern, 'one1two2three3four4'):
        print m.group(),    #1 2 3 4
 
def sub():
    """
    re.sub(pattern, repl, string[, count])
    使用repl替换string中每一个匹配的子串后返回替换后的字符串
    :return:
    """
    pattern = re.compile(r'(\w+)-(\w+)')
    s = 'I-can, hello world!'
    print re.sub(pattern,r'\2 \1', s)   #can I, hello world!
 
def subn():
    """
    re.subn(pattern, repl, string[, count])
    返回 (sub(repl, string[, count]), 替换次数)
    :return:
    """
    pattern = re.compile(r'(\w+)-(\w+)')
    s = 'I-can, hello-world!'
    print re.subn(pattern, r'\2 \1', s)     #('can I, world hello!', 2)
 
 
match()
search()
split()
findall()
finditer()
sub()
subn()

```
