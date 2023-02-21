# 《MYSQL必知必会》笔记

## 第三章：使用MYSQL

```sql
USE database;
SHOW databases;
SHOW tables;
SHOW grants;
```

## 第四章：检索数据

```sql
-- 全选
SELECT * FROM students;
-- 多列
SELECT name, age FROM students;
-- 去重
SELECT DISTINCT score FROM students;
-- 前五条
SELECT name, age FROM students LIMIT 5;
-- 从第三条开始的五条
SELECT name, age FROM students LIMIT 3, 5;
SELECT name, age FROM students LIMIT 5 OFFSET 3;
```

## 第五章：排序检索数据

```sql
-- 按照名字字母顺序排序
SELECT name, age FROM students ORDER BY name;
-- 指定排序方向 得分最高的在前面，降序      升序：ASC，默认就是升序
SELECT name, age FROM students ORDER BY score DESC;
-- 多列排序 DESC关键字只应用到直接位于其前面的列名
SELECT name, age FROM students ORDER BY score DESC, name;
```

## 第六章：过滤数据

> 在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误

```sql
-- 过滤
SELECT name, score FROM students WHERE score > 80;
```

操作符：> >= < <= = BEWTEEN

空值检查：

```sql
SELECT name FROM students WHERE name IS NULL;
```

## 第七章：数据过滤

> SQL（像多数语言一样）在处理OR操作符前，优先处理AND操作符。
> 任何时候使用具有AND和OR操作符的WHERE子句，都应该使用圆括号明确地分组操作符。不要过分依赖默认计算次序

AND操作符：

```sql
SELECT * FROM students WHERE name = stephen AND score = 80;
```

OR操作符：

```sql
SELECT * FROM students WHERE name = stephen OR score = 80;
```

IN操作符：用来指定条件范围，范围中的每个条件都可以进行匹配。

```sql
-- 成绩为70或者80的学生
SELECT * FROM students WHERE score IN (70,80);
```

NOT操作符：

```sql
-- id除了100或者200的学生
SELECT * FROM students WHERE id NOT (100, 200);
```

## 第八章：用通配符进行过滤

%表示任何字符出现任意次数

```sql
-- 名字以ste起头的学生
SELECT * FROM students WHERE name LIKE 'ste%';
-- 名字中包含ste的学生
SELECT * FROM students WHERE name LIKE '%ste%';
-- 名字以s起头e结尾的学生
SELECT * FROM students WHERE name LIKE 's%e';
```

_下划线的用途与%一样，但下划线只匹配单个字符而不是多个字符

## 第九章：用正则表达式进行搜索

基本字符匹配：

```sql
SELECT * FROM students WHERE score REGEXP '70'
```

进行OR匹配：

```sql
SELECT * FROM students WHERE score REGEXP '70|80'
```

匹配几个字符之一：

```sql
SELECT * FROM students WHERE score REGEXP '[678]0'
```

## 第十章：创建计算字段

使用 `Concat()` 函数拼接

```sql
SELECT Concat(vendor_name, '(', vender_country, ')') as vendor_title FROM vendors ORDER BY vendor_name;
```

## 第十一章：使用数据处理函数

日期和时间处理函数

无论你什么时候指定一个日期，不管是插入或更新表值还是用WHERE子句进行过滤，日期必须为格式yyyy-mm-dd

```sql
SELECT * FROM students WHERE Date(birth) = '2010-01-01';
```

## 第十二章：汇总数据

- AVG()
- COUNT()
- MAX()
- MIN()
- SUM()

## 第十三章：分组数据

> GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前

创建分组：

```sql
SELECT Count(*) as stuNum FROM students GROUP BY class_id;
```

过滤分组：

> 目前为止所学过的所有类型的WHERE子句都可以用HAVING来替代。唯一的差别是WHERE过滤行，而HAVING过滤分组

```sql
SELECT Count(*) as stuNum FROM students GROUP BY class_id HAVING COUNT(*) >= 40;
```

> HAVING和WHERE的差别 这里有另一种理解方法，WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤。这是一个重要的区别，WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中基于这些值过滤掉的分组

## 第十四章：使用子查询

> 使用子查询并不总是执行这种类型的数据检索的最有效的方法

利用子查询进行过滤：

```sql
SELECT cust_id FROM orders WHERE order_num IN (SELECT order_num
                                               FROM orderitems
                                               WHERE prod_id = 'TNT2');
```

作为计算字段使用子查询：

```sql
SELECT cust_name,
       cust state,
      (SELECT COUNT(*)
       FROM orders
       WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

## 第十五章：联结表

内部联结：

```sql
SELECT * FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
```

## 第十六章：创建高级联结

在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字指定包括其所有行的表（RIGHT指出的是OUTER JOIN右边的表，而LEFT指出的是OUTER JOIN左边的表）

```sql
-- 为了检索所有客户，包括那些没有订单的客户
SELECT * FROM customers LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id;
```

## 第十七章：组合查询

UNION规则：
1. UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔
2. UNION中的每个查询必须包含相同的列、表达式或聚集函数
3. 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型

包含或取消重复的行：

> UNION从查询结果集中自动去除了重复的行（换句话说，它的行为与单条SELECT语句中使用多个WHERE子句条件一样）

这是UNION的默认行为，但是如果需要，可以改变它。事实上，如果想返回所有匹配行，可使用UNION ALL而不是UNION

对组合结果进行排序

> SELECT语句的输出用ORDER BY子句排序。在用UNION组合查询时，只能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后。实际上MySQL将用它来排序所有SELECT语句返回的所有结果。

## 第十八章：全文本搜索

> 并非所有引擎都支持全文本搜索。两个最常使用的引擎为MyISAM和InnoDB，前者支持全文本搜索，而后者不支持。

一般在创建表时启用全文本搜索：

```sql
CREATE TABLE productnotes
(
	...
	note_text text NULL,
	FULLTEXT(note_text)
)
```

MySQL根据子句FULLTEXT(note_text)的指示对它进行索引。这里的FULLTEXT索引单个列，如果需要也可以指定多个列。

在索引之后，使用两个函数Match()和Against()执行全文本搜索，其中Match()指定被搜索的列，Against()指定要使用的搜索表达式。

```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');
```

Match()和Against()用来建立一个计算列（别名为rank），此列包含全文本搜索计算出的等级值。

使用查询扩展：表中的行越多（这些行中的文本就越多），使用查询扩展返回的结果越好。

## 第十九章：插入数据

插入完整的行：

```sql
INSERT INTO students
(
	stu_name,
	stu_address,
	stu_age
)
VALUES('curry', 'gsw', 34);
```

插入多条数据：

```sql
INSERT INTO students
(
	stu_name,
	stu_address,
	stu_age
)
VALUES('curry', 'gsw', 34), ('james', 'lakers', 37);
```

插入检索出的数据：

INSERT还存在另一种形式，可以利用它将一条SELECT语句的结果插入表中。这就是所谓的INSERT SELECT，顾名思义，它是由一条INSERT语句和一条SELECT语句组成的。

## 第二十章：更新和删除数据

更新数据：

```sql
UPDATE students
SET stu_name = 'curry'
	stu_age = 30
WHERE stu_id = 30;
```

删除数据：

```sql
DELETE FROM students
WHERE stu_id = 10006;
```

## 第二十一章：创建和操纵表

创建表：

```sql
create TABLE students
(
	stu_id int NOT NULL AUTO_INCREMENT,
	stu_name char(50) NOT NULL,
	stu_age int,
	PRIMARY KEY (stu_id)
) ENGINE=InnoDB;
```

更新表：

```sql
ALTER TABLE students
ADD stu_phone char(20);
```

删除表：

```sql
DROP TABLE students;
```

## 第二十二章：使用视图

>视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询

创建视图：

```sql
CREATE VIEW productcustomers AS
SELECT cust_name，cust_contact,prod_idFROM customers,orders，order
items
WHERE customers.cust_id = orders.cust_id
AND orderitems.order num = orders.order num;
```

用视图重新格式化检索的数据：

```sql
CREATE VIEW vendorlocations AS
SELECT Concat(RTrim(vend_name)，' ( '，RTrim(vend_country)，')')
AS vend_title
FROM vendors
ORDER BY vend_name;
```

用视图过滤不想要的数据：

```sql
CREATE VIEw customeremaillist
AS
SELECT cust_id,cust_name,cust_emailFROM customers
WHERE cust_emai7 IS NOT NULL;
```

## 第二十三章：使用存储过程

>存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用不仅限于批处理。

为什么要使用存储过程：
1. 简化复杂操作
2. 保证数据完整性
3. 防止错误
4. 提高性能

执行存储过程：

MySQL称存储过程的执行为调用，因此MySQL执行存储过程的语句为CALL。CALL接受存储过程的名字以及需要传递给它的任意参数。

```sql
CALL productpricing
(
	@pricelow,
	@pricehig,
	@priceaverage
);
```

创建存储过程：

```sql
CREATE PROCEDURE productpricing
BEGIN
	SELECT Avg(prod_price) AS priceaverage 
	FROM products;
END;
```

删除存储过程：

```sql
DROP PROCEDURE productpricing;
```

