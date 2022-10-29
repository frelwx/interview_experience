# Mysql Crash Course

## 第1章 了解SQL

1. 数据库（database）
2. 表（table）
3. 列（column）：表中的一个字段，每个列都有数据类型
4. 行（row）：或者记录
5. 主键（primary key）：一列或一组列，唯一标识每一行。Mysql主键规则：
   1. 任意两行的主键都不相同
   2. 每个行都必须有一个主键，主键不允许是NULL

## 第2章 Mysql简介

1. DBMS：数据库管理软件
2. Mysql分为服务器和客户端，服务器软件为DBMS，客户机可以是命令行、GUI
3. 登录选项：mysql -u ben -p -h myserver -P 9999

## 第3章 使用Mysql

选择数据库

```mysql
use crashcourse;
```

查看数据库

```mysql
show databases;
```

查看表

```mysql
show tables;
```

显示customers表的列信息

```mysql
show columns from customers;
```

可以用以下语句替代

```mysql
describe customers;
```

```mysql
show status; // 用来显示广泛的服务器状态信息
show create database; 和 show create table; 分别用来显示创建特定数据库或表的mysql语句
show grants;用来显示授予用户的安全权限
show errors;和 show warnings;用来显示服务器错误或者警告信息
```

```mysql
help show; // 显示允许的show语句
```

## 第4章 检索数据

select语句，从products表中检索prod_name列

```mysql
SELECT prod_name
FROM products;
```

命令行中sql语句以分号结束。

SQL不区分大小写。最好关键字大写，列和表名使用小写。SQL分成多行更容易阅读

检索多个列，列之间用逗号隔开

```mysql
SELECT prod_id, prod_name, pro_price
FROM products;
```

检索所有列(可以检索出名字未知的列)

```mysql
SELECT * 
FROM products;
```

检索不同的行（对行去重），以下语句只返回vend_id不同的行

```mysql
SELECT DISTINCT vend_id
FROM products;
```

不能部分的使用DISTINCT，该关键字用于所有列而不仅是紧挨着它的列，如果

```mysql
SELECT DISTINCT vend_id, prod_price 
FROM prod_price;
```

那么只有所有的列都相同的时候才算重复

限制结果，以下语句返回前五行

```mysql
SELECT prod_name
FROM products
LIMIT 5;

```

返回从第5行开始的5行（从0开始计数），如果不够则返回只能返回那么多的行

```mysql
LIMIT 5, 5
```

完全限定名

```mysql
products.prod_name 等同于 prod_name; crashcourse.products 等同于 products;
```

## 第5章 排序检索数据

子句（clause）：SQL语句由子句构成，比如SELECT语句的FROM子句、ORDER BY子句等

ORDER BY子句，默认升序，可以指定多个列进行排序，按逗号分隔。指定降序需要使用关键字DESC，该关键字直接应用到前面紧挨着的列名。Mysql默认a和A是相同的排序顺序

```mysql
SELECT prod_name 
FROM products
ORDER BY prod_name;
```

FROM ORDER BY LIMIT 顺序

```mysql
FROM
ORDER BY
LIMIT
```

## 第6章 过滤数据

WHERE子句，位于FROM 之后，ORDER BY 之前

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price = 2.50;
```

某些特殊的检查，BETWEEN, NULL检查

```mysql
WHERE prod_price BETWEEN 5 AND 10;
```

```mysql
WHERE prod_price IS NULL;
```

## 第7章 数据过滤

在where子句中使用 AND，OR，IN ，NOT操作符

```mysql
SELECT pro_id, prod_price, prod_name
FROM products
WHERE vend_id = 1003 AND prod_price <= 10;
```

可以用圆括号改变优先级，AND 优先级比OR 高

IN 操作符，功能和OR相当，不过比IN更快，而且可以与其他SELECT语句组合使用

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id IN(1002, 1003)
ORDER BY prod_name;
```

NOT否定一个条件，Mysql支持NOT对IN、BETWEEN、EXISTS子句取反

## 第8章 用通配符进行过滤

通配符（wildcard）

搜索模式（search pattern）由字面值、通配符或两者组合而成的搜索条件

使用LIKE操作符

%通配符：任何字符出现任意次数 ，但是无法匹配NULL

```mysql
SELECT prod_name, prod_id
FROM products
WHERE prod_name LIKE 'jet%';
```

下划线_通配符：匹配任意单个字符

通配符放在开头，搜索速度是最慢的

## 第9章 用正则表达式进行搜索

使用REGEXP替代LIKE

LIKE 如果没有使用通配符，执行精确匹配，必须每一个字符都相等才返回。而=会忽略尾部的空格

正则表达式如果没有通配符，则返回包含当前字符串的行

点（.）：匹配任意一个字符

(|)：进行OR匹配，表示匹配其中之一

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000|2000'
ORDER BY prod_name;
```

([])：匹配其中之一，是另一种形式的OR匹配

```mysql
SELECT prod_name 
FROM products
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;
```

[123] Ton 是 [1|2|3] Ton的缩写，方括号不可以省略，否则会有优先级问题（1|2| Ton）

此外，可以在集合中的开头加入^表示匹配除了这些字符以外的任何字符

```mysql
([^1|2|3])
```

还可以使用范围匹配

```mysql
[1-5],[a-z]
```

使用两个反斜杠匹配特殊字符

```mysql
\\-;
\\.
\\\;匹配反斜杠本身
```

（为什么是两个？Mysql自己解释一个，正则表达式库解释另一个）

匹配字符类（P58）

```mysql
[:alnum:] 任意字母或数字同[a-zA-Z0-9]
[:alpha:] 任意字符同[a-zA-Z]
```

使用时外面要再加一层中括号，即

```mysql
[[:alnum:]]
```

正则表达式重复元字符

```mysql
* 0个或多个匹配
+ 1个或多个匹配，等于{1, }
? 0个或1个匹配，等于{0, 1}
{n} 指定数目的匹配
{n,} 不少于指定数目的匹配
{n,m} 匹配数目范围内的匹配，m不超过255
```

重复元字符作用于前面一个字符

定位符

前面的匹配都是匹配任意一个位置的字符

定位元字符

```mysql
^ 文本的开始
$ 文本的结尾
[[:<:]] 词的开始
[[::>]] 词的结尾
'^ok';表示以ok为开始的字符
'ok$';表示以ok为结尾的字符
```

^有两种用法，用在[]集合开始，或者用在表达式开始

用^开始每个表达式，用$结束每个表达式，可以使REGEXP和LIKE的作用一样

在不使用测试正则表达式

```mysql
SELECT 'hello' REGEXP '[0-9]';因为hello中没有数字，因此无法匹配
```

## 第10章 计算字段

字段（field）：基本和列的意思相同

Concat()函数

```mysql
SELECT Concat(vend_name, '(', vend_country, ')')
FROM vendors
ORDER BY vend_name;
```



```mysql
RTrim() 去掉串右边的空格
LTrim() 去掉串左边的空格
Trim()  去掉串两边的空格
```

使用别名（alias），又称导出列（derived column）

```mysql
SELECT Concat(vend_name, '(', vend_country, ')') AS
vend_title
FROM vendors
ORDER BY vend_name;
```

使用计算字段，支持+-*/

```mysql
SELECT prod_id, 
	   quantity
	   item_price,
	   quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

使用SELECT测试函数和计算的办法

```mysql
SELECT 2*3;
SELECT Trime('abc ')
```

## 第11章 使用数据处理函数

文本处理函数

```mysql
Left() 返回字符串左边的字符
Length() 返回串的长度
...
```

日期和时间处理函数

```mysql
Date() 返回日期和时间的日期部分
Year() 返回日期的年份
```

比如

```mysql
SELECT cust_id, order_num
FROM orders
WHERE Date(orders_date) = '2005-09-01';
```

BETWEEN AND 也可以用于时间

数值处理函数，在不同的DBMS中是统一的

## 第12章 汇总数据

聚集函数（aggregate function）：运行在行组上，计算和返回单个值的函数

```mysql
AVG()
Count() 返回某列的行数
MAX()
MIN()
SUM()
```



```mysql
SELECT AVG(prod_price) AS ave_price
FROM products;
```



使用Count(*) 对表中行的数目进行统计，不管NULL还是非NULL都一样统计

使用Count(column)对特定列具有值的行进行计数，忽略NULL

MAX()和MIN()忽略NULL，用于文本时，分别返回最后一行、第一行



以上五个聚集函数都可以使用

ALL参数：对所有行进行计算

DISTINCT参数： 对不同值进行计算

```mysql
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;
```

DISTINCT必须用于列名，不能用于*

## 第13章 分组数据

创建分组，使用GROUP BY子句，位于WHERE 之后，ORDER BY之前

```mysql
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;
```

过滤分组

WHERE过滤行，HAVING过滤分组

WHERE在分组之前进行过滤，HAVING在分组之后过滤

```mysql
SELECT vend_id COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;
```

分组和排序

分组必须使用选择的列或表达式列

```mysql
SELECT order_num, SUM(quantity*item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50
ORDER BY ordertotal;
```

SELECT 子句次序

```mysql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

## 第14章 使用子查询

查询（query）：任何SQL语句都是查询，但一般指SELECT

利用子查询进行过滤（用在where中）

```mysql
SELECT cust_id
FROM customers
WHERE order_num IN (SELECT order_num 
                    FROM orderitems
                    WHERE prod_id = 'TNT2');
```

子查询，检索包含物品TNT2的所以订单号

外查询，检索子查询中的订单号对应的客户id

注意，在WHERE中使用子查询，SELECT语句选出的列数目必须和WHERE的列数目匹配



作为计算字段使用子查询

```mysql
SELECT cust_name,
	   cust_state,
	   (SELECT COUNT(*)
        FROM orders
        WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

此处子查询，对于customers中的每一行（即customers.cust_id），都去计算一次子查询。

## 第15章 联结表

联结（join）

vendors表的主键又叫做products表的外键

外键（foreign key）外键为某个表中的一列，它包含另一个表的主键值

创建联结

```mysql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name;
```

WHERE 子句匹配vendors表中的vend_id和products表中的vend_id

笛卡尔积（Cartesian product），又称叉联结（cross join）：没有联结条件的表关系返回笛卡尔积，行的数目是两个表的行数之积。

应该保证所有联结都有where子句，通过where语句可以消除不匹配的行

内部联结、等值联结（equijoin）：基于两个表之间的相等测试。

可以用如下语法替代

```mysql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id
```

联结多个表

```mysql
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
	AND orderitems.prod_id = products.prod_id
	AND order_num = 20005;
```

首先筛选订单号为20005号的订单中的物品。orderitems引用products表中的prod_id。这些物品通过vend_id联结道vendors表中的供应商。

## 第16章 创建高级联结

使用表别名

```mysql
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
	AND oi.order_num = o.order_num
	AND prod_id = 'TNT2'
```

不同类型的联结

自联结，原因：在一条SELECT中可能不止一次引用相同的表

```mysql
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
                 FROM products
                 WHERE prod_id = 'DTNTR');
                 
                 
SELECT prod_id, prod_name
FROM products as p1, products as p2
WHERE p1.vend_id = p2.vend_id
	AND p2.prod_id = 'DTNTR';
```

自然联结:排除重复的列，使得每个列只返回一次。需要自己完成，可以对其他表指定列名，剩下的表使用*

目前为止，我们建立的每个列都是自然联结的 

```mysql
SELECT c.*, o.order_num, o.order_date, oi.prod_id, oi.quantity, oi.item_price
FROM customers as c, orders as o, orderitems as oi
WHERE c.cust_id = o.cust_id
WHERE oi.order_num = o.order_num
	AND prod_id = 'FB'
```

外部联结：

有时候需要包含没有关联行的那些行

以下SQL检索所有客户，包括那些没有订单的客户

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id
```

语法和使用INNER JOIN的内部联结相似，但是外部联结还包括没有关联的行。

必须使用LEFT或者RIGHT指定包括其所有行的表，RIGHT指的是OUTER JOIN右边的表，LEFT相反。

左外部联结可以颠倒FROM或者WHERE子句中表的顺序转换为右外部联结

使用带聚集函数的联结

## 第17章 组合查询

将多条SELECT语句的结果组合或并（union）起来

union可以用where中的多个条件替代

创建组合条件

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_id IN (1001, 1002);
```

UNION的每个查询列必须相同，但次序不需要相同

UNION默认合并重复的行

使用UNION ALL可以返回所有匹配的行

对组合结果进行排序

只能在组后一行用ORDER BY，对所有结果进行排序

## 第18章 全文本搜索

INnoDB引擎不支持全文本搜搜，MyISAM支持

为了使用全文本搜索，必须索引被搜索的列

主要依靠MATCH()和AGAINST()两个方法

启动全文本搜索支持需要在创建表的时候为列名使用FULLTEXT()函数修饰

也可以在插入所有数据后再修改FULLTEXT()，这样比先FULLTEXT()，再一个个插入数据建立索引快

```mysql
SELECT note_text
FROM productnotes
WHERE MATCH(note_text) AGAINST('rabbit')
```

MATCH的值必须和FULLTEXT()的相同，并且顺序正确

返回值按照匹配程度排序

可以在SELECT 中使用

```mysql
SELECT note_text MATCH(note_text) AGAINST('rabbit') AS ranks
FROM productnotes;
```

使用查询扩展，在AGAINST('key' WITH QUERY EXPANSION )

使用布尔文本搜索

```mysql
SELECT note_text
FROM productnotes
WHERE MATCH('note_text') AGAINST('heavy' IN BOOLEAN MODE);
```



## 第19章 插入数据

插入一行

插入多行

插入select的结果

## 第20章 更新和删除数据

更新数据，一旦省略where，将更新所有的行

```mysql
UPDATE customers
SET cust_name = 'The Fudds',
	cust_email = 'elmer@fudd.com'
WHERE cust_id = 1005;
```

可以配合SELECT使用

删除数据，(删除的是行，删除列请用UPDATE)

```mysql
DELETE 
FROM customers
WHERE cust_id = 1006;
```

## 第21章 创建和操纵表

创建表

```mysql
CREATE TABLE customers(
	cust_id int NOT NULL AUTO_INCREMENT
	cust_name, char(50), NOT NULL
)ENGINE=InnoDB;
```

可以使用PRIMARY KEY()说明主键

每个表只允许一个AUTO_INCREMENT列，而且它必须被索引

可以为每一列指定默认值

引擎类型

InnoDB 是可靠的事务处理引擎，不支持全文本搜索

MyISAM 不支持事务处理，但支持全文本搜索，

MEMORY 在功能上等同于MyISAM，但数据存储在内存中，速度很快



更新表

增加一个列

```mysql
ALTER TABLE vendors
ADD vend_phone char(20);
```

删除列

```mysql
ALTER TABLE vendors
DROP COLUMN vend_phone;
```

删除表

```mysql
DROP TABLE customers2;
```

重命名表

```mysql
RENAME TABLE customers2 TO customers
```

## 第22章 使用视图

视图本身不包含数据，可以把它看成一个虚拟表

创建视图

CREATE VIEW AS

SELECT .....

如果在使用视图的时候也有ORDER BY，那视图的ORDER BY会覆盖定义时候的

使用视图的时候的WHERE将和定义时的WHERE 组合在一起



更新视图等价于更新其基表，因此，更新视图有很多限制

## 第23章 使用存储过程

存储过程：一条或多条Mysql语句

执行存储过程

```mysql
CALL productpricing(@pricelow, @pricehigh, @priceaverage);
```

创建存储过程

```mysql
CREATE PROCEDURE productpricing()
BEGIN 
	SELECT Avg(prod_price) AS priceaverage
	FROM products;
END;
```

删除存储过程

```mysql
DROP PROCEDURE productpricing;
```

创建存储过程中可以使用变量

OUT型代表返回值

IN型代表传入存储过程的值

INOUT

使用SELECT x INTO y进行赋值

所有变量在调用的时候必须用@修饰

智能存储过程

使用DECLARE 定义局部变量

使用SHOW CREATE PROCEDURE xxx;查看建立存储过程的方法

查看某个存储过程的详细信息

SHOW PROCEDURE STATUS [LIKE XXX]

## 第24章 使用游标

游标（cursor）：Mysql游标只能用于存储过程和函数，存储过程完成后，游标就消失

创建游标

```mysql
CREATE PROCEDURE productpricing()
BEGIN
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
END;
```

打开、关闭游标

```mysql
OPEN ordernumbers;
CLOSE ordernumbers;
```

在处理open语句时执行查询，存储检索出的数据以供浏览和检索

使用游标数据，FETCH，每次取一行，然后将指针往下移动一行

```mysql
CREATE PROCEDURE productpricing()
BEGIN
	DECLARE o INT;
	
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	OPEN ordernumbers;
	FETCH ordernumbers INTO o;
	CLOSE ordernumbers;
END;
```

## 第25章 使用触发器

触发器是Mysql响应以下任意语句而自动执行的一条sql语句（或者位于BEGIN 和 END之间的一组语句）

- DELETE
- INSERT
- UPDATE

只有表才支持触发器，视图和虚拟表都不支持

每个表的每个事件前后最多定义一个触发器，也就是一共6个

如果BEFORE触发器失败，则对应语句不执行

创建触发器

```mysql
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'PRoduct added';
```

该触发器在每行插入完成后输出一句话'PRoduct added'

删除触发器

```mysql
DROP TRIGGER XXX;
```

INSERT 触发器

- 触发器代码内可以使用一个叫NEW的虚表，访问被插入的行
- 在触发器中，可以更改NEW的值
- 对于AUTO_INCREMENT列，NEW在执行之前包含0，在INSERT执行之后包含新的自动生成值

DELETE 触发器

- 在触发器代码内可以使用一个叫OLD的虚表，访问被删除的行
- OLD是只读的

UPDATE 触发器

- 在触发器代码内，可以访问OLD获得被更改前的值，使用NEW访问新的更新的值
- OLD只读
- 在触发器中，可以更改NEW，即更改将要用于UPDATE中的值

触发器中不支持CALL

## 第26章 管理事务处理

事务(transaction)：一组sql语句

事务处理可以保证一组sql语句完整的执行，或者完全不执行

标识事务的开始

```mysql
START TRANSACTION
```

回退（ROLLBACK）：撤销指定sql语句的过程

ROLLBACK会回退 START TRANSACTION 之后的所有语句

ROLLBACK只对INSERT UPDATE DELETE 起作用，回退SELECT没有意义

不能回退CREATE和DROP，可以用，但不会起作用



提交（COMMITE）：将未存储的sql语句写入到数据库表中

一般的sql语句都是implicit commit的、

但是事务处理中需要显示提交

当COMMIT或者ROLLBACK后，事务会自动关闭

保留点（savepoint）：事务处理中设置的临时占位符，可以回退到此处

创建占位符

```mysql
SAVEPOINT xxx;
ROLLBACK TO xxx;
```

## 第27章 全球化和本地化

字符集

编码

校对：决定比较方式，在排序、分组时很重要

## 第28章 安全管理

管理用户：Mysql的用户账号和信息放在一个名为mysql的表中

```mysql
USE mysql;
SELECT user 
FROM user;
```

访问mysql数据库的user表的user列

创建用户账号

```mysql
CREATE USER ben IDENTIFIED BY 'PASSWORD'
```

不建议通过在user中插入一行的方式建立用户

重命名用户

```mysql
RENAME USER ben TO bforta;
```

删除用户

```mysql
DROP USER bforta;
```

查看访问权限

```mysql
SHOW GRANTS FOR bforta;
```

设置权限

```mysql
GRANT SELECT ON crashcourse.* TO bforta;
```

允许用户在crashcourse的所有表上使用SELECT;

撤销权限

```mysql
REVOKE SELECT ON crashcourse.* TO bforta;
```

更改密码

```mysql
SET PASSWORD FOR bforta = Password('new password')
```

Password用来对密码加密

不指定用户名的时候，默认为当前登录的用户