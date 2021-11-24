# 了解SQL  

- 数据库（database） 保存有组织的数据的容器（通常是一个文件或一组文件）
- 表（table） 某种特定类型数据的结构化清单  
- 模式（schema） 关于数据库和表的布局及特性的信息  
- 列（column） 表中的一个字段。所有表都是由一个或多个列组成的
- 数据类型（datatype） 所容许的数据的类型。每个表列都有相应的数据类型，它限制（或容许）该列中存储的数据   
- 行（row） 表中的一个记录  
- 主键（primary key）一列（或一组列），其值能够唯一区分表中每个行   
  - 任意两行都不具有相同的主键值  
  - 每个行都必须具有一个主键值（主键列不允许NULL值）  

SQL 是结构化查询语言（Structured Query Language）的缩写。 SQL是一种专门用来与数据库通信的语言。  

# 使用MySQL  

选择数据库：

```
USE <数据库名称>
```

查看可用数据库的一个列表：

```
SHOW DATABASE;
```

获得一个数据库内的表的列表：

```
SHOW TABLES;
```

显示表列：

```
SHOW COLUMNS FROM <表名称>;
DESCRIBE <表名称>;
```

显示服务器错误或警告消息：

```
SHOW ERRORS;
SHOW WARNINGS;
```

# 检索数据

检索单个列：

```
SELECT <列名称> FROM student;
```

检索多个列：

```
SELECT <列名称1>,<列名称2>,<列名称3>,... FROM student;
```

检索所有列：

```
SELECT * FROM student;
```

**一般，除非你确实需要表中的每个列，否则最好别使用*通配符。虽然使用通配符可能会使你自己省事，不用明确列出所需列，但检索不需要的列通常会降低检索和应用程序的性能。**  

检索不同的行：

```
SELECT DISTINCT <列名称> FROM student;
```

**不能部分使用DISTINCT DISTINCT关键字应用于所有列而不仅是前置它的列。如果给出SELECT DISTINCT column1,column2，除非指定的两个列都不同，否则所有行都将被检索出来。    **

限制结果：

```
SELECT <列名称> FROM student LIMIT <限制行数>;
```

返回不超过"限制数量"的行数。

```
SELECT <列名称> FROM student LIMIT <偏移行数>,<限制行数>;
SELECT <列名称> FROM student LIMIT <限制行数> OFFSET <偏移行数>;
```

**LIMIT中指定要检索的行数为检索的最大行数。如果没有足够的行， MySQL将只返回它能返回的那么多行。**

  











