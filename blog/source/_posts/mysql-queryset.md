---
title: Mysql必知必会笔记
date: 2018-09-16 19:02:58
tags:
  - Mysql
categories:
  - Mysql
---

最近看了一下mysql基础知识， 主要集中在相关查询操作上，记录如下：

其中使用的数据库结构和数据为http://forta.com/books/0672327120/ 中的SQL脚本
#### 计算次序
```
SELECT * FROM products WHERE vend_id=1002 or vend_id =1003;
SELECT * FROM products WHERE (vend_id=1002 or vend_id=1003) AND prod_price >=10;
SELECT * FROM products WHERE vend_id not in (1002, 1003) ORDER BY prod_name;
SELECT * FROM products ;
SELECT * FROM products WHERE prod_name like '_ ton anv%';	# _ 只匹配单个字符而不是多个字符
```
#### 正则表达式
```
SELECT * FROM products ;
SELECT * FROM products WHERE prod_name REGEXP '.000' ORDER BY prod_name;	# 匹配任意一个字符
SELECT * FROM products WHERE prod_name regexp '1000|2000';
SELECT * FROM products WHERE prod_name regexp '[123] ton';
```
#### 创建计算字段
拼接： 将值联结到一起构成单个值， 使用Concat()函数来来拼接两个列, 函数中为多个字符串
```
SELECT CONCAT(vend_name, ' (', vend_country, ') ') AS vend_title FROM vendors order by vend_name;
```
执行算术计算
```
SELECT prod_id, quantity, item_price, quantity*item_price as total_price FROM orderitems WHERE order_num=20005;
```

#### 使用数据处理函数
```
日期和时间处理函数
ADDDATE(expr,days)	添加一个日期(天，周等)
ADDTIME(expr1,expr2)	添加一个时间(时，分等)
CURDATE()	返回当前日期
CURTIME()	返回当前时间
DATE(expr)	返回日期时间的日期部分
DATEDIFF(expr1,expr2)	计算两个日期之差
DATE_ADD(date,INTERVAL expr unit)	高度灵活的日期运算函数
DATE_FORMAT(date,format)	返回一个格式化的日期或时间串
DAY(date)	返回一个日期的天数部分
DAYOFWEEK(date)	对于一个日期，返回对应的星期几
HOUR(time)	返回一个时间的小时部分
MINUTE(time)	返回一个时间的分钟部分
MONTH(date)	返回一个日期的月份部分
NOW()		返回当前日期和时间
SECOND(time)	返回一个时间的秒部分
TIME(expr)	返回一个日期时间的时间部分
YEAR()		返回一个日期的年份部分
 
数值处理函数
ABS(X)	返回一个数的绝对值
COS(X)	返回一个角度的余弦
EXP(X)	返回一个数的指数值
MOD(N,M)	返回除操作的余数
PI()	返回圆周率
RAND()	返回一个随机数
SIN(X)	返回一个角度的正弦
SQRT(X)	返回一个数的平方根
TAN(X)	返回一个角度的正切
 
SELECT * FROM orders;
SELECT * from orders WHERE DATE(order_date) = '2005-09-01';
SELECT * FROM orders WHERE DATE(order_date) BETWEEN '2005-09-01' and '2005-10-01';
SELECT * from orders where YEAR(order_date)=2005 and MONTH(order_date) = 10;
```

#### 汇总数据

- 聚合函数
    运行在行组上，计算和返回单个值的函数

```
AVG([DISTINCT] expr)	返回某列的平均值
COUNT(expr)	返回某列的行数
MAX(expr)	返回某列的最大值
MIN(expr)	返回某列的最小值
SUM(expr)	返回某列值之和
 
 
SELECT AVG(prod_price) AS avg_price FROM products WHERE vend_id=1003;
SELECT COUNT(*) FROM customers;
SELECT COUNT(cust_email) FROM customers;
SELECT SUM(item_price* quantity) FROM orderitems WHERE order_num=20005;
-- 集合不同值 DISTINCT
SELECT AVG(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id=1003;
SELECT COUNT(*) AS num_items, MIN(prod_price), MAX(prod_price), AVG(prod_price) FROM products;
```
#### 分组数据
- 分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算
- WHERE 过滤行， HAVING 过滤分组
- WHERE 在数据分组前进行过滤，having在数据分组后进行过滤
- GROUP BY子句可以包含任何数目的列
- 如果在group by子句中嵌套了分组， 数据将在最后规定的分组上进行汇总
- GROUP BY子句中列出的每个列都必须是检索列或有效的表达式(单不能是聚集函数)
- SELECT 语句中的每个列都必须在group by 子句中给出
- 如果分组列中具有null值，则null将作为一个分组返回，如果列中有多行null值，它们将分为一组
- GROUP BY 子句必须出现在where子句之后，order by子句之前
- 使用 with rollup关键字， 可以得到每个分组汇总的值

```
SELECT * FROM orders;
SELECT vend_id, COUNT(*) as num FROM products GROUP BY vend_id WITH ROLLUP;
SELECT cust_id,COUNT(*) as order_count FROM orders GROUP BY cust_id HAVING order_count >= 2;
SELECT * FROM products;
SELECT vend_id,COUNT(*) FROM products WHERE prod_price >=10 GROUP BY vend_id HAVING COUNT(*) >=2;
```

SELECT子句及其顺序

子句 | 说明|是否必须使用|
---|---|---|
SELECT | 要返回的列或表达式| 是
FROM | 从中检索数据的表| 仅在从表选择数据时使用
WHERE| 行级过滤| 否
GROUP BY| 分组说明| 仅在按组计算聚集时使用
HAVING| 组级过滤| 否
ORDER BY| 输出排序顺序| 否
LIMIT| 要检索的行数| 否

```
SELECT order_num, SUM(quantity*item_price) AS total  FROM orderitems GROUP BY order_num HAVING total >=50 ORDER BY total;
```

#### 使用子查询
子查询：嵌套在其他查询中的查询
子查询进行过滤
```
SELECT * FROM orderitems WHERE prod_id='TNT2';
SELECT * FROM orders WHERE order_num in (20005, 20007);
SELECT cust_id FROM orders WHERE order_num in (SELECT order_num FROM orderitems WHERE prod_id='TNT2');
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (SELECT cust_id FROM orders WHERE order_num in (SELECT order_num FROM orderitems WHERE prod_id='TNT2'));
```
计算字段使用子查询

```
SELECT COUNT(*) AS orders FROM orders WHERE cust_id=10001;

SELECT cust_name, cust_state, (SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders FROM customers ORDER BY cust_name;
```
#### 联结表查询
- 内部联结： 基于两个表之间的相等测试

```
SELECT vend_name, prod_name, prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
SELECT prod_name, vend_name, prod_price, quantity FROM orderitems, products, vendors WHERE products.vend_id = vendors.vend_id AND orderitems.prod_id = products.prod_id AND order_num=20005;
SELECT cust_name, cust_contact FROM customers, orders, orderitems WHERE customers.cust_id = orders.cust_id AND orderitems.order_num = orders.order_num AND prod_id = 'TNT2';
```
#### 组合查询
利用union操作符将多条select组合成一个结果集
使用场景：
    
    - 在单个查询中从不同的表返回类似结构的数据
    - 对单个表执行多个查询，按单个查询返回数据
 规则：
 
 - SELECT 语句之间使用UNION关键字连接
 - UNION每个查询必须包含相同的列、表达式或者聚集函数
 - 列数据类型必须兼容，类型不必完全相同，但必须是DBMS可以隐含地转换类型
 - UNION 从查询结果中自动去除了重复的行，如果需要返回所有匹配行，可以使用UNION ALL来展示
 - UNION 只能使用一条ORDER BY 子句，必须出现在最后一条SELECT语句之后,作用于所有SELECT语句返回的结果
 
```
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price<=5;
SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002);
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <=5 UNION ALL SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001,1002);
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price<=5 OR vend_id IN (1001, 1002);
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price<=5 UNION SELECT  vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001,1002) ORDER BY vend_id,prod_price; 
```

#### 全文本搜索
- MyISAM引擎支持全文本搜索，InnoDB不支持
- MATCH()指定被搜索的列
- Against()指定要使用的搜索表达式

```
SELECT * FROM productnotes WHERE MATCH(note_text) Against('rabbit');
SELECT * FROM productnotes WHERE note_text like "%rabbit%";
SELECT * FROM productnotes WHERE MATCH(note_text) Against('anvils');
```

#### 视图
视图是虚拟的表， 视图只包含使用时动态检索数据的查询

视图功能
    
    - 重用sql语句
    - 简化复杂的sql操作
    - 使用表的组成部分而不是整个表
    - 保护数据
    - 更改数据格式和表示 
基本语句
    
    - 创建使用 CREATE VIEW
    - 查看创建视图的语句 SHOW CREATE VIEW viewname
    - 删除视图， DROP VIEW viewname
```
SELECT cust_name, cust_contact FROM customers, orders, orderitems WHERE customers.cust_id = orders.cust_id AND 
orderitems.order_num = orders.order_num AND prod_id="TNT2";
-- 创建视图
CREATE VIEW productcustomers AS SELECT cust_name, cust_contact, prod_id FROM customers, orders, orderitems WHERE customers.cust_id=orders.cust_id AND
orderitems.order_num = orders.order_num;
SELECT cust_name, cust_contact FROM productcustomers;
SELECT cust_name, cust_contact FROM productcustomers WHERE prod_id='TNT2';
-- 重新格式化检索出的数据
CREATE VIEW vendorlocations AS SELECT CONCAT(RTRIM(vend_name), '(', RTRIM(vend_country), ')') AS vend_title FROM vendors ORDER BY vend_name;
SELECT * FROM vendorlocations;
-- 视图过滤不想要的数据
CREATE VIEW customeremiallist AS SELECT * FROM customers WHERE cust_email IS NOT NULL;
SELECT * FROM customeremiallist;
-- 使用视图与计算字段
CREATE VIEW orderitemsexpanded AS SELECT order_num, prod_id, quantity, item_price, quantity*item_price AS expanded_price FROM orderitems;
SELECT * FROM orderitemsexpanded WHERE order_num=20005;
-- 查看视图
SHOW CREATE VIEW orderitemsexpanded;
```
