# R Package(DBI)

## **Introduction**

### **DBI–RMySQL**

#### R语言和关系型数据库的两种配合方式

> * 用SQL来提取需要的数据，存为文本再由R来读入。
> * 将R与外部数据库连接，直接在R中操作数据库。

#### 连接方式的两种选择：

> * ODBC方式，需要安装RODBC包并安装ODBC驱动(个人感觉，这种方法相对DBI方式不太方便，所以不做过多介绍)
> * DBI方式，可以根据已经安装的数据库类型来安装相应的驱动

### DBI (Database Interface)

> 提供通用接口和每个数据库管理系统(DBMS)

> * DBI包实现了一系列的数据库接口函数，这些函数独立于实际上存储数据的数据库服务器。只需要在最初和数据库建立连接时指出用户将应用哪个通信接口。当改变数据库时，你只要改变一条命令即可(该命令指出你希望通信的数据库)。为了获取这种独立性，用户需要安装其他的R包来处理不同的数据库的通信细节。对于常用的数据库，R有很多专有的数据库包。这里为了和存储在服务器上的MySQL数据库通信，我们需要安装RMySQL包.



## Usage

|                              |                                                          |
| ---------------------------- | -------------------------------------------------------- |
| 建立本地连接                       | channel = dbConnect(MySQL(), dbname, username, password) |
|                              | summary(channel)                                         |
|                              | dbGetInfo(channel)                                       |
|                              | dbListResults(channel)                                   |
|                              | dbGetException(channel)                                  |
|                              | show(channel)                                            |
|                              |                                                          |
| 查看数据库的表                      | dbListTables(channel)                                    |
| 获取整个name表数据                  | dbReadTable(channel, name)                               |
| 获取name表的变量名                  | dbListFields(channle, name)                              |
| 将R中的数据导出到MySQL数据库            | dbWriteTable(channel, name, value)                       |
| dbRemoveTable(channel, name) | 删除MySQL数据库中的name表                                        |
|                              |                                                          |
| Query using DBI              | res = dbSendQuery(conn, statement)                       |
|                              | dbGetStatement(res)                                      |
|                              | dbColumnInfo(res)                                        |
|                              | dbGetRowsAffected(res)                                   |
|                              | dbGetRowCount(res)                                       |
|                              | dbHasCompleted(res)                                      |
|                              | dbGetInfo(res)                                           |
|                              | dbClearResult(res)                                       |
|                              |                                                          |
| transactions: DBMS事务管理       |                                                          |
| 提交事务(数据库中的数据发生变化)            | dbCommit(conn)                                           |
| 设置事务起始点(数据库中的数据原始状态)         | dbBegin(conn)                                            |
| 返回事务起始点(数据库中的数据不发生变化)        | dbRollback(conn)                                         |
| 关闭连接                         | dbDisconnect(channel)                                    |

### Build Connection and Read Data

```
## 载入DBI包
library(DBI)
## 与服务器建立连接，这里的连接是MySQL类型的，也可以是其他类型的服务器
  ## 例如SQL Sever...，这里我们应用一个给定读取数据权限的Web数据库，用户名和主机名已经给定。
ucscDb <- dbConnect(RMySQL::MySQL(), user = "genome", host = "genome-mysql.cse.ucsc.edu")

## 向连接的服务器发出查询请求，要求列出连接到服务器中的所有数据库
databases <- dbGetQuery(conn = ucscDb, statement = "show databases;")

##断开与服务器的连接
dbDisconnect(ucscDb)

## 列出查询结果，这里就是列出了该服务器中的所有数据库
head(databases)
dim(databases)
```

```
## 对其中的数据库hg1g进行探索，那就建立与数据库hg19的连接
hg19 = dbConnect(RMySQL::MySQL(), dbname = "hg19", 
                 user = "genome", host = "genome-mysql.cse.ucsc.edu")

## 列出hg19中的所有数据表,并查看所有表的个数
allTables <- dbListTables(hg19)
## table个数
length(allTables)
head(allTables)

## 列出数据库hg19中表`affyU133Plus2`的所有变量名及数据的观测值个数
dbListFields(hg19, "affyU133Plus2")
dbGetQuery(hg19, "select count(*) from affyU133Plus2")

## 读取数据表affyU133Plus2
affyData = dbReadTable(hg19, "affyU133Plus2")
str(affyData)

## 断开与数据库的连接
dbDisconnect(hg19)
```

### Query

* dbSendQuery() 将SQL查询发送到DBMS并返回结果对象。 该查询仅限于SELECT语句。 如果要发送其他语句，例如INSERT，UPDATE，DELETE等，使用dbSendStatement()
* dbFetch() 与dbSendQuery() 返回的结果对象一起调用。 它还接受一个参数，该参数指定要返回的行数，例如 n =200。如果要获取所有行，请使用n = -1。&#x20;
* 完成检索数据后，将调用dbClearResult()。 它释放与结果对象关联的资源。

```
library(DBI)

con <- dbConnect(
  RMariaDB::MariaDB(),
  host = "relational.fit.cvut.cz",
  port = 3306,
  username = "guest",
  password = "relational",
  dbname = "sakila"
)

res <- dbSendQuery(con, "SELECT * FROM film WHERE rating = 'G'")
df <- dbFetch(res, n = 3)
dbClearResult(res)

head(df, 3)

##############################################################################
  film_id            title
1       2   ACE GOLDFINGER
2       4 AFFAIR PREJUDICE
3       5      AFRICAN EGG
                                                                                                            description
1                  A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China
2                          A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank
3 A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico
  release_year language_id original_language_id rental_duration rental_rate length replacement_cost
1         2006           1                   NA               3        4.99     48            12.99
2         2006           1                   NA               5        2.99    117            26.99
3         2006           1                   NA               6        2.99    130            22.99
  rating               special_features         last_update
1      G        Trailers,Deleted Scenes 2006-02-15 04:03:42
2      G Commentaries,Behind the Scenes 2006-02-15 04:03:42
3      G                 Deleted Scenes 2006-02-15 04:03:42
##############################################################################
```

### Read part of a table from a database

如果数据集很大，则可能一次要获取有限数量的行。 如下所示，这可以通过使用while循环来实现，在循环中，函数dbHasCompleted() 用于检查正在进行的行，而dbFetch() 与n = X参数一起使用，指定每次迭代要返回多少行。 同样，我们在最后调用dbClearResult() 释放资源。

```
res <- dbSendQuery(con, "SELECT * FROM film")
while (!dbHasCompleted(res)) {
  chunk <- dbFetch(res, n = 300)
  print(nrow(chunk))
}
```

### Quoting queries

DBI支持两种方法来避免来自用户提供的参数的SQL注入攻击：引用和参数化查询&#x20;

* Quoting 使用函数dbQuoteLiteral（）执行参数值的引用，该函数支持许多R数据类型，包括日期和时间. 如果用户可能要提供表名或列名以用于查询中以进行数据检索，则这些名称或标识符也必须转义。 由于可能存在转义这些标识符的特定于DBMS的规则，因此DBI提供了函数dbQuoteIdentifier（）来生成安全的字符串表示形式。

```
safe_id <- dbQuoteIdentifier(con, "rating")
safe_param <- dbQuoteLiteral(con, "G")

query <- paste0("SELECT title, ", safe_id, " FROM film WHERE ", safe_id, " = ", safe_param )
query
## [1] "SELECT title, `rating` FROM film WHERE `rating` = 'G'"

res <- dbSendQuery(con, query)
dbFetch(res)

##              title rating
## 1   ACE GOLDFINGER      G
## 2 AFFAIR PREJUDICE      G
## 3      AFRICAN EGG      G
## Showing 3 out of 178 rows.

dbClearResult(res)

## The same result can be had by using glue::glue_sql(). 
   对查询字符串内大括号之间出现的任何变量或R语句执行相同的安全引用
id <- "rating"
param <- "G"
query <- glue::glue_sql("SELECT title, {`id`} FROM film WHERE {`id`} = {param}", .con = con)
df <- dbGetQuery(con, query)
head(df, 3)
```

### Parameterized queries

DBI支持两种方法来避免来自用户提供的参数的SQL注入攻击：引用和参数化查询&#x20;

* 与其自己执行参数替换，不如通过在查询中包含占位符将其推入DBMS。 不同的DBMS使用不同的占位符方案，DBI通过SQL表达式传递。占位符Placeholders 仅适用于文字值。查询的其他部分，例如表或列标识符，仍然需要用dbQuoteIdentifier（）引起来。对于单个参数集，可以使用dbSendQuery（）或dbGetQuery（）的params参数。它接受一个列表，其成员被替换为查询中的占位符。

```
params <- list("G")
safe_id <- dbQuoteIdentifier(con, "rating")
query <- paste("SELECT * FROM film WHERE ", safe_id, " = ?")
query   ## [1] "SELECT * FROM film WHERE  `rating`  = ?"
res <- dbSendQuery(con, query, params = params)
dbFetch(res, n = 3)

## Perform the same query with different sets of parameter values, dbBind() is used. 
res <- dbSendQuery(con, "SELECT * FROM film WHERE rating = ?")
dbBind(res, list("G"))
dbFetch(res, n = 3)

dbBind(res, list("PG"))
dbFetch(res, n = 3)

## dbBind() can be used to execute the same statement with multiple values at once.
res <- dbSendQuery(con, "SELECT * FROM film WHERE rating = ?")
dbBind(res, list(c("G", "PG")))
dbFetch(res, n = 3)

dbClearResult(res)
dbDisconnect(con)
```

### SQL data manipulation

基础数据库的SQL查询（例如UPDATE，DELETE，INSERT INTO和DROP TABLE），DBI提供了两个功能。&#x20;

* dbExecute（）将SQL语句传递给DBMS以执行，并返回受影响的行数。&#x20;
* dbSendStatement（）以相同的方式执行，但是返回结果对象。 用结果对象调用dbGetRowsAffected（）以获取受影响的行的计数。 然后，您需要随后使用结果对象调用dbClearResult（）来释放资源。

```
library(DBI)
con <- dbConnect(RSQLite::SQLite(), ":memory:")

dbWriteTable(con, "cars", head(cars, 3))

dbExecute(
  con,
  "INSERT INTO cars (speed, dist) VALUES (1, 1), (2, 2), (3, 3)"
)
```

```
rs <- dbSendStatement(
  con,
  "INSERT INTO cars (speed, dist) VALUES (4, 4), (5, 5), (6, 6)"
)
dbGetRowsAffected(rs)
dbClearResult(rs)
dbReadTable(con, "cars")
dbDisconnect(con)
```



