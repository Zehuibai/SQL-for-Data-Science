# 04 Subqueries and Joins in SQL

## Subqueries

### Introduction

查询（query） 任何SQL 语句都是查询。但此术语一般指SELECT 语句。

假如需要列出订购物品RGAN01 的所有顾客

```
SELECT order_num
FROM OrderItems
WHERE prod_id = 'RGAN01';

SELECT cust_id
FROM Orders
WHERE order_num IN (20007,20008);

## 结合这两个查询
SELECT cust_id
FROM Orders
WHERE order_num IN (SELECT order_num
                FROM OrderItems
                WHERE prod_id = 'RGAN01');
```

在SELECT 语句中，子查询总是从内向外处理。在处理上面的SELECT 语 句时，DBMS 实际上执行了两个操作。首先，它执行下面的查询：SELECT order\_num FROM orderitems WHERE prod\_id='RGAN01' 此查询返回两个订单号：20007 和20008。然后，这两个值以IN 操作符要求的逗号分隔的格式传递给外部查询的WHERE 子句。外部查询变成： SELECT cust\_id FROM orders WHERE order\_num IN (2007,2008)&#x20;

* **注意：只能是单列:** 作为子查询的SELECT 语句只能查询单个列。企图检索多个列将返回 错误。

### Using subqueries as calculated fields

假如需要显示Customers 表中 每个顾客的订单总数。订单与相应的顾客ID 存储在Orders 表中。要对每个顾客执行COUNT(\*)，应该将它作为一个子查询。

```
SELECT cust_name,
cust_state,
(SELECT COUNT(*)
FROM Orders
WHERE Orders.cust_id = Customers.cust_id) AS orders
FROM Customers
ORDER BY cust_name;
```

![](<.gitbook/assets/image (3).png>)

## Join

### Create

创建联结非常简单，指定要联结的所有表以及关联它们的方式即可。

```
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id;
```

* 没有WHERE 子句，第一个表中的每一行将与第二个表中的每一行配对，而不管它们 逻辑上是否能配在一起。
* 笛卡儿积（**cartesian product**） 由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数目 将是第一个表中的行数乘以第二个表中的行数。

### Inner Join

目前为止使用的联结称为等值联结（equijoin），它基于两个表之间的相 等测试。这种联结也称为内联结（inner join）。其实

```
SELECT vend_name, prod_name, prod_price
FROM Vendors
INNER JOIN Products ON Vendors.vend_id = Products.vend_id;
```

联结多个表

```
SELECT prod_name, vend_name, prod_price, quantity
FROM OrderItems, Products, Vendors
WHERE Products.vend_id = Vendors.vend_id
AND OrderItems.prod_id = Products.prod_id
AND order_num = 20007;
```



### Aliases&#x20;

```
SELECT cust_name, cust_contact
FROM Customers AS C, Orders AS O, OrderItems AS OI
WHERE C.cust_id = O.cust_id
AND OI.order_num = O.order_num
AND prod_id = 'RGAN01';
```

### Self Joins

自联结通常作为外部语句，用来替代从相同表中检索数据的使用子查 询语句。虽然最终的结果是相同的，但许多DBMS 处理联结远比处理 子查询快得多。应该试一下两种方法，以确定哪一种的性能更好。

```
SELECT cust_id, cust_name, cust_contact
FROM Customers
WHERE cust_name = (SELECT cust_name
FROM Customers
WHERE cust_contact = 'Jim Jones');

## Equal
SELECT c1.cust_id, c1.cust_name, c1.cust_contact
FROM Customers AS c1, Customers AS c2
WHERE c1.cust_name = c2.cust_name
AND c2.cust_contact = 'Jim Jones';
```

### Natural Join

无论何时对表进行联结，应该至少有一列不止出现在一个表中（被联结的列）。标准的联结（前一课中介绍的内联结）返回所有数据，相同的列 甚至多次出现。自然联结排除多次出现，使每一列只返回一次。 怎样完成这项工作呢？答案是，系统不完成这项工作，由你自己完成它。 自然联结要求你只能选择那些唯一的列，一般通过对一个表使用通配符 （SELECT \*），而对其他表的列使用明确的子集来完成。

```
SELECT C.*, O.order_num, O.order_date,
OI.prod_id, OI.quantity, OI.item_price
FROM Customers AS C, Orders AS O,
OrderItems AS OI
WHERE C.cust_id = O.cust_id
AND OI.order_num = O.order_num
AND prod_id = 'RGAN01';
```

### Outer Join

在使用OUTER JOIN 语法时，必须使用RIGHT 或LEFT 关键字指定包括其所有行的表 （RIGHT 指出的是OUTER JOIN 右边的表，而LEFT 指出的是OUTER JOIN 左边的表）。上面的例子使用LEFT OUTER JOIN 从FROM 子句左边的表 （Customers 表）中选择所有行。为了从右边的表中选择所有行，需要使 用RIGHT OUTER JOIN

总是有两种基本的外联结形式：左外联结和右外联结。它们 之间的唯一差别是所关联的表的顺序。换句话说，调整FROM 或WHERE 子句中表的顺序，左外联结可以转换为右外联结。因此，这两种外联 结可以互换使用，哪个方便就用哪个。

### Joins with Aggregate Functions

```
SELECT Customers.cust_id,
COUNT(Orders.order_num) AS num_ord
FROM Customers
INNER JOIN Orders ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;
```

![](<.gitbook/assets/image (4).png>)



## Compound query: Union

### Union

多数SQL 查询只包含从一个或多个表中返回数据的单条SELECT 语句。 但是，SQL 也允许执行多个查询（多条SELECT 语句），并将结果作为一 个查询结果集返回。这些组合查询通常称为并（union）或复合查询 （compound query）。使用UNION 很简单，所要做的只是给出每条SELECT 语句，在各条语句 之间放上关键字UNION。

```
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';

SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';

## Combined
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state IN ('IL','IN','MI')
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';
```

* 使用UNION 组合SELECT 语句的数目，SQL 没有标准限制。但是，最 好是参考一下具体的DBMS 文档，了解它是否对UNION 能组合的最大 语句数目有限制。
* UNION 必须由两条或两条以上的SELECT 语句组成，语句之间用关键字 UNION 分隔（因此，如果组合四条SELECT 语句，将要使用三个UNION 关键字）。
* UNION 中的每个查询必须包含相同的列、表达式或聚集函数（不过， 各个列不需要以相同的次序列出）。
* UNION 的列名: 如果结合UNION 使用的SELECT 语句遇到不同的列名，那么会返回什 么名字呢？比如说，如果一条语句是SELECT prod\_name，而另一条 语句是SELECT productname，那么查询结果返回的是第一个名字，举的这个例子就会返回prod\_name. 。这也意味着你可以对第一个名字使用别名， 因而返回一个你想要的名字。
* SELECT 语句的输出用ORDER BY 子句排序。在用UNION 组合查询时，只 能使用一条ORDER BY 子句，它必须位于最后一条SELECT 语句之后。对 于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一 部分的情况，因此不允许使用多条ORDER BY 子句。

### Duplicate lines

* 返回 所有的匹配行，可使用UNION ALL
* UNION 从查询结果集中自动去除了重复的行
* UNION 几乎总是完成与多个WHERE 条件相同 的工作。UNION ALL 为UNION 的一种形式，它完成WHERE 子句完成 不了的工作。如果确实需要每个条件的匹配行全部出现（包括重复行）， 就必须使用UNION ALL，而不是WHERE。
*





