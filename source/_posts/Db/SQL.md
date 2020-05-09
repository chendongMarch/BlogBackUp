---
layout: post
title: SQL
categories:
  - Db
tags:
  - Database
  - SQL
keywords:
  - Database
  - SQL
abbrlink: 1299261529
date: 2017-04-13 00:00:00
---

## 1. 前言

SQL，指结构化查询语言，全称是(Structured Query Language)，SQL 语句是大小写不敏感的。  

- 说明：
	- 使用`db_name`作为数据库名，使用`tb_name`作为表名。
	- 使用`col_name`作为列名，使用`row_name`作为行名。
	- 使用`alias_name`作为别名。  
	- [xxx]表示可选使用的属性。
	- (a...|b...|c...)表示三种情况任选一种使用。

<!--more-->

## 2. 数据库配置
```bash
编辑 ./bash_profile 文件，加入别名

open ~/.bash_profile

# mysql
alias mysql=/usr/local/mysql/bin/mysql

连接到mysql服务
mysql -u root -p
```

## MySQL

```bash
sudo /usr/local/mysql/support-files/mysql.server start
sudo /usr/local/mysql/support-files/mysql.server stop
sudo /usr/local/mysql/support-files/mysql.server restart
```

## 3. 库操作语句

```sql
显示数据库列表
SHOW databases;

创建数据库
CREATE DATABASE db_name;  
 
使用数据库
USE db_name;
  
删除数据库
DROP database db_name;

使用字符集
SET names utf8; 
```
 
## 4. 表操作语句
关于表字段约束的讲解见附2  

```sql
创建表
CREATE TABLE table_name(
col_name1 data_type(size) [约束],
col_name2 data_type(size) [约束],
col_name3 data_type(size) [约束],
....
);

eg:
CREATE TABLE Persons(
PersonID int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
);


删除表
DROP TABLE tb_name;

删除表数据，但是保留表结构
TRUNCATE TABLE tb_name

增加一列
ALTER TABLE tb_name ADD col_name 数据类型 [约束]

删除一列
ALTER TABLE tb_name DROP COLUMN col_name;

修改一列
ALTER TABLE tb_name ALTER old_col_name new_col_name 数据类型 [约束]

修改一列的数据类型
/* SQL Server / MS Access */
ALTER TABLE tb_name
MODIFY COLUMN col_name 数据类型
/* My SQL / Oracle */
ALTER TABLE tb_name
ALTER COLUMN col_name 数据类型

添加主键
ALTER TABLE tb_name ADD PRIMARY KEY(col_name);

删除主键
ALTER TABLE tb_name DROP PRIMARY KEY(col_name);
```



## 5. 增删改查语句
### 5.1 SELECT 语句
```sql
基本查询语句
[DISTINCT] 可选，用于返回唯一不同的值。
SELECT [DISTINCT] * FROM tb_name;
SELECT [DISTINCT] col_name1,col_name2 FROM tb_name;
eg:
SELECT name,country FROM Websites;
```
### 5.2 SELECT INTO 语句
使用`SELECT INTO`和 `INSERT INTO SELECT` 语句，复制表数据。
`MySQL` 数据库不支持 `SELECT INTO` 语句，但支持 `INSERT INTO  SELECT`。  
新表将会使用 `SELECT` 语句中定义的列名称和类型进行创建。您可以使用 `AS` 关键字来应用新名称。  

```sql
复制全部数据或者使用WHERE子句筛选
SELECT *
INTO new_tb_name [IN db_name]
FROM old_tb_name;
or
复制指定列数据或者使用WHERE子句筛选
SELECT col_name...
INTO new_tb_name [IN db_name]
FROM old_tb_name;

复制全部数据或者使用WHERE子句筛选
INSERT INTO new_tb_name
SELECT * FROM old_tb_name;
or
复制指定列数据或者使用WHERE子句筛选
INSERT INTO new_tb_name
(col_name...)
SELECT col_name...
FROM old_tb_name;
```

### 5.3 INSERT INTO 语句
```sql
// 不指定列名插入
INSERT INTO tb_name
VALUES (value1,value2,value3,...);

// 指定列名插入
INSERT INTO tb_name (col_name1,col_name2,col_name3,...)
VALUES (value1,value2,value3,...);

eg:
INSERT INTO Websites (name, url, country)
VALUES ('stackoverflow', 'http://stackoverflow.com/', 'IND');
```

### 5.4 UPDATE 语句
```sql
使用UPDATE语句时，一定要添加WHERE条件，否则会更新所有的数据。

UPDATE tb_name
SET col_name1=value1,col_name2=value2,...
WHERE (这里参照WHERE子句，匹配指定数据);

eg:
UPDATE Websites 
SET alexa='5000', country='USA' 
WHERE name='aasdfghjkl';
```

### 5.5 DELETE 语句
```sql
DELETE FROM tb_name
WHERE (这里参照WHERE子句，匹配指定数据);

DELETE * FROM tb_name
WHERE (这里参照WHERE子句，匹配指定数据);

eg:
DELETE FROM Websites
WHERE name='百度' AND country='CN';
```


## 6. WHERE 子句
```sql
SELECT col_name1,col_name2
FROM tb_name
WHERE col_name1 operator value1 
[逻辑运算符 col_name2 operator value2...];
```

### 6.1 逻辑运算符
```sql  
逻辑运算符
NOT 非操作
AND 与操作，表达式前后条件必须都成立才为true
OR 或操作，表达式前后操作有一个成立即为true

逻辑运算的优先级：() NOT AND OR

eg:
SELECT * FROM Websites
WHERE alexa > 15
AND (country='CN' OR country='USA');
```

### 6.2 比较运算符
```sql
/*
比较运算符
=,>,<,>=,<=,
<>(不等于，一些版本中写作!=),
BETWEEN...AND(在范围内),
LIKE(模糊查询),
IN(指定所有可能值进行匹配),
IS NULL(空值判断)
*/

eg:
空值查询 IS NULL
SELECT * FROM tb)nae 
WHERE col_name IS NULL;

存在查询 IN
SELECT * FROM tb_name 
WHERE col_name 
IN (5000,3000,1500);
SELECT * FROM tb_name 
WHERE col_name 
IN ('abc','tyu','test');


区间查询 BETWEEN...AND...
区间查询两边都是闭区间，类似[1,100]
SELECT * FROM tb_name 
WHERE col_name 
BETWEEN 100 AND 200;


模糊查询 LIKE
SELECT * FROM tb_name 
WHERE col_name LIKE 'M%';
todo 模糊查询通配符
% 表示多个字值，_ 下划线表示一个字符。  
M% : 为能配符，正则表达式，表示的意思为模糊查询信息为 M 开头的。  
%M% : 双百分号表示查询的信息在内容中间。  
%M_% : 表示查询的字母在内容的倒数第二位。
```


## 7. JOIN 子句(表连接)
`JOIN` 子句用于基于这些表之间的共同字段把来自两个或多个表的行结合起来。
 
|连接方式|描述|
|:--|:--|
|INNER JOIN|如果表中有至少一个匹配，则返回行|
|LEFT JOIN|即使右表中没有匹配，也从左表返回所有的行|
|RIGHT JOIN|即使左表中没有匹配，也从右表返回所有的行| 
|FULL JOIN|只要其中一个表中存在匹配，则返回行|


### 7.1 INNER JOIN 内连接
`INNER JOIN` 也可以简写为 `JOIN`
`INNER JOIN `关键字在表中存在至少一个匹配时返回行。
```sql
SELECT col_name
FROM tb_name1
INNER JOIN tb_name2
ON tb_name1.col_name=tb_name2.col_name;
```


### 7.2 LEFT OUTER JOIN 左外连接
`LEFT OUTER JOIN `也可以简写为 `LEFT JOIN`
`LEFT JOIN` 关键字从左表返回所有的行，即使右表中没有匹配，如果右表中没有匹配，则结果为 `NULL`。
```sql
SELECT col_name
FROM tb_name1
LEFT OUTER JOIN tb_name2
ON tb_name1.col_name=tb_name2.col_name;
```

### 7.3 RIGHT OUTER JOIN 右外连接
`RIGHT OUTER JOIN` 也可以简写为 `RIGHT JOIN`
`RIGHT JOIN` 关键字从右表返回所有的行，即使左表中没有匹配，如果左表中没有匹配，则结果为` NULL`。
```sql
SELECT col_name
FROM tb_name1
RIGHT OUTER JOIN tb_name2
ON tb_name1.col_name=tb_name2.col_name;
```


### 7.4 FULL OUTER JOIN 全外连接
`FULL OUTER JOIN` 关键字只要左表 和右表 其中一个表中存在匹配，则返回行.
`FULL OUTER JOIN` 关键字结合了 `LEFT JOIN` 和 `RIGHT JOIN` 的结果。
```sql
SELECT col_name
FROM tb_name1
FULL OUTER JOIN tb_name2
ON tb_name1.col_name=tb_name2.col_name;
```

## 8. 关键字
### 8.1 AS 关键字,使用别名
使用`AS`关键字，可以为表名称或列名称指定别名，基本上，创建别名是为了让列名称的可读性更强，以下情况使用别名很有用。

- 在查询中涉及超过一个表
- 在查询中使用了函数
- 列名称很长或者可读性差
- 需要把两个列或者多个列结合在一起

ps:别名如果包含空格，要求使用双引号或方括号。

```sql
列别名用法
使用列别名查询之后的展示数据将使用别名来展示
SELECT col_name AS alias_name
FROM tb_name;
eg:
SELECT name AS n, country AS c
FROM Websites;


表别名用法
SELECT col_name
FROM tb_name AS alias_name;
eg:
SELECT w.name, w.url, a.count, a.date 
FROM Websites AS w, access_log AS a 
WHERE a.site_id=w.id and w.name="百度";
```

### 8.2 ORDER BY 关键字
```sql
ORDER BY 关键字默认按照升序对记录进行排序。
如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。

SELECT col_name1,col_name2
FROM tb_name
ORDER BY col_name (ASC|DESC)[,col_name (ASC|DESC)...];

eg:
SELECT * FROM Websites
ORDER BY alexa DESC;
```

### 8.3 LIMIT 关键字
```sql
使用limit关键字，可以跳过m条数据，查询n条数据，m可以省略，表示从头开始查询。
SELECT * FROM tb_name LIMIT [m,]n;

eg：
SELECT * FROM Websites LIMIT 3,2;
```


### 8.4  UNION 关键字
`UNION` 操作符用于合并两个或多个 `SELECT` 语句的结果集。  
`UNION` 内部的每个 `SELECT` 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 `SELECT` 语句中的列的顺序必须相同。  
`UNION` 结果集中的列名总是等于 `UNION` 中第一个 `SELECT` 语句中的列名。  
`UNION` 只会选取不同的值。请使用 `UNION ALL` 来选取重复的值！  

```sql
SELECT col_name  FROM tb_name1
UNION
SELECT col_name  FROM tb_name2;

SELECT col_name  FROM tb_name1
UNION ALL
SELECT col_name  FROM tb_name2;
```



## 9. 函数

### 9.1 CONCAT()连接函数
结果将会拼接`CONCAT()`函数中的全部值

```sql  
下面的例子将会拼接三个字段的值(结果：'www.baidu.com,100,china')，并作为一列(site_info)显示

SELECT name, CONCAT(url, ', ', alexa, ', ', country) AS site_info
FROM Websites;
```


## 10. 索引
在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。在表中创建索引，以便更加快速高效地查询数据。
用户无法看到索引，它们只能被用来加速搜索/查询。
更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。
### 10.1 创建索引
```sql
创建一个简单的索引。允许使用重复的值：
CREATE INDEX index_name
ON table_name (column_name)

在表上创建一个唯一的索引。不允许使用重复的值：唯一的索引意味着两个行不能拥有相同的索引值。 
CREATE UNIQUE INDEX index_name
ON table_name (column_name)
注释：用于创建索引的语法在不同的数据库中不一样。因此，检查您的数据库中创建索引的语法。
 
eg:
创建索引
CREATE INDEX PIndex
ON Persons (LastName)
在多个列上创建索引
CREATE INDEX PIndex
ON Persons (LastName, FirstName)
```

### 10.2 删除索引
```sql
/* MS Access */
DROP INDEX index_name ON table_name
/* MS SQL Server */
DROP INDEX table_name.index_name
/* DB2/Oracle */
DROP INDEX index_name
/* MySQL */
ALTER TABLE table_name DROP INDEX index_name
```




## 附1. 模糊查询
### 附1.1. 通配符
|通配符|描述|示例|
|:--|:--|:--|
|%|代替0个或者多个字符|chen%,匹配chen开头的全部数据|
|_|代替一个字符|ch_n,匹配类似chan,chbn,chcn这种数据|
|[charlist]|字符序列中的任一个单个字符|[ABC],匹配A,B,C|
|[^charlist] or [!charlist]|不在字符序列中的任一个字符|[!ABC],匹配除了A,B,C以外的其他字符|
```sql
SELECT col_name
FROM tb_name
WHERE col_name LIKE pattern;
```

### 附1.2 正则表达式
```sql
todo
```

## 附2. 表约束
|约束|描述|
|:--|:---|
|NOT NULL|指示某列不能存储 NULL 值。|
|UNIQUE |保证某列的每行必须有唯一的值。|
|PRIMARY KEY |NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。|
|FOREIGN KEY | 保证一个表中的数据匹配另一个表中的值的参照完整|性。
|CHECK | 保证列中的值符合指定的条件。|
|DEFAULT |规定没有给列赋值时的默认值。|

### 附2.1 NOT NULL 非空约束
`NOT NULL` 约束强制列不接受` NULL `值。  
`NOT NULL` 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。  

```sql
CREATE TABLE Persons(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
);
```

### 附2.2 UNIQUE 唯一约束
`UNIQUE` 约束唯一标识数据库表中的每条记录。  
`UNIQUE `和 `PRIMARY KEY` 约束均为列或列集合提供了唯一性的保证。  
`PRIMARY KEY` 约束拥有自动定义的 `UNIQUE` 约束。  
请注意，每个表可以有多个 `UNIQUE` 约束，但是每个表只能有一个 `PRIMARY KEY` 约束。
  
```sql
/* MySQL */
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (P_Id)
)

/* SQL Server / Oracle / MS Access */
CREATE TABLE Persons
(
P_Id int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)


命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
)
```
更改`UNIQUE`约束
```sql 
ALTER TABLE Persons
ADD UNIQUE (P_Id)
如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)

撤销 UNIQUE 约束 
MySQL：
ALTER TABLE Persons
DROP INDEX uc_PersonID
SQL Server / Oracle / MS Access：
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```

### 附2.3 PRIMARY KEY 主键约束
`PRIMARY KEY` 约束唯一标识数据库表中的每条记录。
主键必须包含唯一的值。
主键列不能包含 `NULL` 值。
每个表都应该有一个主键，并且每个表只能有一个主键。

```sql
/* MySQL */
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (P_Id)
)

/* SQL Server / Oracle / MS Access */
CREATE TABLE Persons
(
P_Id int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)


命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束 
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
)
注释：在上面的实例中，只有一个主键 PRIMARY KEY（pk_PersonID）。然而，pk_PersonID 的值是由两个列（P_Id 和 LastName）组成的。
```
更改`PRIMARY KEY`约束
```sql
ALTER TABLE Persons
ADD PRIMARY KEY (P_Id)
如需命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束，请使用下面的 SQL 语法：
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
注释：如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）。


撤销 PRIMARY KEY 约束
/* MySQL */
ALTER TABLE Persons
DROP PRIMARY KEY
/* SQL Server / Oracle / MS Access */
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```


### 附2.4 FOREIGN KEY 外键约束
一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。
`FOREIGN KEY` 约束用于预防破坏表之间连接的行为。
`FOREIGN KEY` 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

例如：   
我们现在有`Persons`表用来存储用户信息，`Persons`表主键`P_Id`表示用户id，`Orders`表用来存储用户订单，`Orders`表外键`P_Id`指向 `Persons`表的主键`P_Id`，来约束`Orders`表中所有订单的用户id必须是在用户表中出现的。
  `Orders` 表中的 `P_Id` 列指向 `Persons` 表中的 `P_Id` 列。
  `Persons` 表中的 `P_Id` 列是 `Persons` 表中的 `PRIMARY KEY`。
 `Orders` 表中的 `P_Id` 列是 `Orders` 表中的 `FOREIGN KEY`。   

```sql
/* MySQL */
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
PRIMARY KEY (O_Id),
FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
)

/* SQL Server / Oracle / MS Access */
CREATE TABLE Orders
(
O_Id int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
P_Id int FOREIGN KEY REFERENCES Persons(P_Id)
)


命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
PRIMARY KEY (O_Id),
CONSTRAINT fk_PerOrders FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)
)
```
更改外键约束

```sql 
ALTER TABLE Orders
ADD FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)

如需命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束，请使用下面的 SQL 语法：
ALTER TABLE Orders
ADD CONSTRAINT fk_PerOrders
FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)

撤销 FOREIGN KEY 约束 
/* MySQL */
ALTER TABLE Orders
DROP FOREIGN KEY fk_PerOrders
/* SQL Server / Oracle / MS Access */
ALTER TABLE Orders
DROP CONSTRAINT fk_PerOrders
```

### 附2.5  CHECK 约束
`CHECK` 约束用于限制列中的值的范围。
如果对单个列定义 `CHECK` 约束，那么该列只允许特定的值。
如果对一个表定义 `CHECK` 约束，那么此约束会基于行中其他列的值在特定的列中对值进行限制。
```sql
/* MySQL */
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (P_Id>0)
)

/* SQL Server / Oracle / MS Access */
CREATE TABLE Persons
(
P_Id int NOT NULL CHECK (P_Id>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)


如需命名 CHECK 约束，并定义多个列的 CHECK 约束 
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
)
```
更改`CHECK`约束
```sql
ALTER TABLE Persons
ADD CHECK (P_Id>0)

如需命名 CHECK 约束，并定义多个列的 CHECK 约束
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')

撤销 CHECK 约束 
/* MySQL */
ALTER TABLE Persons
DROP CHECK chk_Person
/* SQL Server / Oracle / MS Access */
ALTER TABLE Persons
DROP CONSTRAINT chk_Person
```


### 附2.6 DEFAULT 约束
`DEFAULT` 约束用于向列中插入默认值。
如果没有规定其他的值，那么会将默认值添加到所有的新记录。
```sql
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)

通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
OrderDate date DEFAULT GETDATE()
)
```
更改`DEFAULT`约束
```sql
添加 DEFAULT 约束
/* MySQL */：
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
/* SQL Server / MS Access */
ALTER TABLE Persons
ALTER COLUMN City SET DEFAULT 'SANDNES'
/* Oracle */
ALTER TABLE Persons
MODIFY City DEFAULT 'SANDNES'



撤销 DEFAULT 约束 
/* MySQL */
ALTER TABLE Persons
ALTER City DROP DEFAULT
/*  SQL Server / Oracle / MS Access */
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```

### 附2.7 AUTO INCREMENT


















