# Introduction

## SQL

SQL是Structured Query Language（结构 化查询语言）的缩写。SQL 是一种专门用来与数据库沟通的语言

### SQL queries <a href="#sql_queries" id="sql_queries"></a>

SQL queries are the most common and essential SQL operations. Via an SQL query, one can search the database for the information needed. SQL queries are executed with the “SELECT” statement. An SQL query can be more specific, with the help of several clauses:

* FROM - it indicates the table where the search will be made.
* WHERE - it's used to define the rows, in which the search will be carried. All rows, for which the WHERE clause is not true, will be excluded.
* ORDER BY - this is the only way to sort the results in SQL. Otherwise, they will be returned in a random order.

```
SELECT * FROM
WHERE active
ORDER BY LastName, FirstName
```

### SQL data manipulation <a href="#sql_data_manipulation" id="sql_data_manipulation"></a>

Data manipulation is essential for SQL tables - it allows you to modify an already created table with new information, update the already existing values or delete them.

With the INSERT statement, you can add new rows to an already existing table. New rows can contain information from the start, or can be with a NULL value.

```
INSERT INTO phonebook(phone, firstname, lastname, address) VALUES('+1 123 456 7890', 'John', 'Doe', 'North America');
```

With the UPDATE statement, you can easily modify the already existing information in an SQL table.



```
UPDATE phonebook SET address = 'North America', phone = '+1 123 456 7890' WHERE firstname = 'John' AND lastname = 'Doe';
```

With the DELETE statement you can remove unneeded rows from a table.

```
DELETE FROM phonebook WHERE WHERE firstname = 'John' AND lastname = 'Doe';
```



****

## Retrieve data

### Select

* 多条SQL 语句必须以分号（；）分隔。多数DBMS 不需要在单条SQL 语句后加分号，但也有DBMS 可能必须在单条SQL 语句后加上分号
* SQL 语句不区分大小写，因此SELECT 与select 是相同的
* 在处理SQL 语句时，其中所有空格都被忽略。SQL 语句可以写成长长 的一行，也可以分写在多行
* 在选择多个列时，一定要在列名之间加上逗号，但最后一个列名后不 加
* **检索所有列**: SELECT 语句还可以检 索所有的列而不必逐个列出它们。在实际列名的位置使用星号（\*）通配 符可以做到这点，SELECT \* FROM Products
* **检索不同的值,** 使用DISTINCT 关键字, SELECT DISTINCT vend\_id FROM Products;
* **限制结果**: 可以用TOP 关键字来限制最多返回多 少行，SELECT TOP 5 prod\_name FROM Products;
  * 如果你使用的是DB2，就得使用下面这样的DB2 特有的SQL 语句：SELECT prod\_name FROM Products FETCH FIRST 5 ROWS ONLY;
  * 如果你使用Oracle，需要基于ROWNUM（行计数器）来计算行，SELECT prod\_name FROM Products WHERE ROWNUM <=5;
  * 如果你使用MySQL、MariaDB、PostgreSQL 或者SQLite，需要使用LIMIT 子句，
    * SELECT prod\_name FROM Products LIMIT 5;&#x20;
    * LIMIT 5 OFFSET 5 指示MySQL 等DBMS 返回从第5 行起的5 行数据。
    * 注意：第0 行 第一个被检索的行是第0 行，而不是第1 行。因此，LIMIT 1 OFFSET 1 会检索第2 行，而不是第1 行。
* 很多DBMS 都支持各种形式的注释语法
  * 注释使用-- （两个连字符）嵌在行内。-- 之后的文本就是注释，
  * 另一种形式的行内注释（但这种形式有些DBMS 不支持) ##



### Sort

