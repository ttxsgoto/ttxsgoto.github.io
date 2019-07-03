---
title: 运维 Awk、Uniq、Sort基本用法
date: 2017-02-07 22:38:32
tags:
  - Awk
  - Sort
  - Uniq
categories:
  - 运维
---

今天对awk、uniq、sort三个命令做了基础功能的学习，这里自己记录一下：

#### 1.awk
数据处理工具，将一行分割成多个“字段”来处理

    awk -F '文本切割符'‘{处理过程}’ 文件名称

如：cat /ect/passwd | awk -F ':' '{print $1}'   #以：分割，打印第一列的数据；如果为$0表示整个文件内容

    cat /etc/passwd | awk -F ''BEGIN {print "begin,goto"} {print $1","$7} END {print “end,end”}  #以空白符作为分割，打印begin，goto开始执行信息，打印1，7行，以end,end结束


#### 2.uniq 
命令用来过滤重复部分显示文件内容,这个命令读取输入文件，并比较相邻的行
```
参数：

-c  显示输出中，在每行行首加上本行在文件中出现的次数
-d  只显示重复行
-u  只显示文件中不重复的各行
-n  前n个字段与每个字段前的空白一起被忽略
+n  前n个字符被忽略，之前的字符被跳过（字符从0开始编号）
-f   n与-n相同，这里n是字段数
-s   n与+n相同，这里n是字符数
常用： uniq -c  首行显示文件中出现的次数
```

#### 3.sort 排序
```
参数：
-u  在输出行中去除重复行
-r   默认的排序方式为升序，-r转换为降序排列
-n  默认按照字符来排序出现10在2前面，-n以数值来排序
-t   后面设定间隔符
-k  指定列数
如：sort -n -k 2 -t ： test.txt  以数值排序，按照第二列以“：”间隔来排列顺序
```
以上简单总结而已，后续如有其他使用，再更新！！！

#### 4.日常使用相关

- 截取日志中特定时间段的日志内容
    
        sed -n '/2016-08-25 09:44:10/,/2016-08-25 09:44:30/p'  1.txt > test.txt


- nginx 访问日志统计访问的url

    截取特定时间段的日志

        cat nginx.acc.log | egrep "12/Aug/2016" | sed -n '/14:59:44/,/15:47:23/p' > a.txt

- 排序

        cat a.txt |awk -F '+0800' '{print $2}'| awk -F ' ' '{print $5}' | sort #对数据进行(ASCII)排序  

- 去重，uniq -c 只会合并相邻的记录，所以在使用它之前，应该先进行排序

        cat a.txt |awk -F '+0800' '{print $2}'| awk -F ' ' '{print $5}' | sort | uniq -c
    
- 再排序，得到 (次数  内容)的文件， sort -k 1 -n -r 指定对第一行进行排序，-n 数字排序，以降序排列

        cat a.txt |awk -F '+0800' '{print $2}'| awk -F ' ' '{print $5}' | sort | uniq -c | sort -k 1 -n -r

