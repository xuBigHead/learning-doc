# 语法

**SQL主要分成四部分**：
（1）数据定义。（SQL DDL）用于定义SQL模式、基本表、视图和索引的创建和撤消操作。
（2）数据操纵。（SQL DML）数据操纵分成数据查询和数据更新两类。数据更新又分成插入、删除、和修改三种操作。
（3）数据控制。包括对基本表和视图的授权，完整性规则的描述，事务控制等内容。
（4）嵌入式SQL的使用规定。涉及到SQL语句嵌入在宿主语言程序中使用的规则。



# DCL

## 概念

DCL（Data Control Language）数据库控制语言  授权，角色控制等。



## 语法

#### GRANT 授权

#### REVOKE 取消授权



# DDL
## 概念

**DDL**（Data Definition Language）数据库定义语言

>  statements are used to define the database structure or schema.



用于定义数据库的三级结构，包括外模式、概念模式、内模式及其相互之间的映像，定义数据的完整性、安全控制等约束。DDL不需要commit。



## 语法
#### 索引相关

##### 查询索引

```mysql
# 查询指定表的索引
SHOW INDEX FROM it_blog;
```



查询结果：

| Table   | Non_unique | Key_name              | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
| ------- | ---------- | --------------------- | ------------ | ----------- | --------- | ----------- | -------- | ------ | ---- | ---------- | ------- | ------------- | ------- | ---------- |
| it_blog | 0          | PRIMARY               | 1            | id          | A         | 2141        |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 0          | uk_url                | 1            | url         | A         | 2172        |          |        |      | BTREE      |         | url唯一索引   | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 1            | user_id     | A         | 2           |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 2            | tag_id      | A         | 4           |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 3            | classify_id | A         | 8           |          |        |      | BTREE      |         |               | YES     |            |



##### 添加索引

```mysql
# 添加普通索引
ALTER TABLE `it_blog` ADD KEY `idx_user`(`user_id`);
CREATE INDEX `idx_user` ON `it_blog`(`user_id`);
# 添加联合索引
ALTER TABLE `it_blog` ADD KEY `idx_user_classiyf`(`user_id`, `classify_id`);
CREATE INDEX `idx_user_classiyf` ON `it_blog`(`user_id`, `classify_id`);
# 添加唯一索引
ALTER TABLE `it_blog` ADD UNIQUE KEY `uk_url`(`url`);
CREATE UNIQUE INDEX `uk_url` ON `it_blog`(`url`);
# 添加主键索引
ALTER TABLE `it_blog` ADD PRIMARY KEY `temp`(`id`);
## 移除主键并设置新的主键
ALTER TABLE `it_blog` DROP PRIMARY KEY, ADD PRIMARY KEY `it_blog`(`id`) USING BTREE;
```



##### 删除索引

```mysql
# 删除索引
ALTER TABLE `it_blog` DROP INDEX `uk_url`;
# 删除索引
DROP INDEX `idx_user` ON `it_blog`;
# 删除主键索引
ALTER TABLE `it_blog` DROP PRIMARY KEY;
```



##### 注意

- 如果单纯移除自增主键而不设置新的主键时，会执行失败。因为表中只能有一列自增列并且改自增列必须要是主键。此时会返回如下结果：

```
ALTER TABLE `it_blog` DROP PRIMARY KEY
> 1075 - Incorrect table definition; there can be only one auto column and it must be defined as a key
> 时间: 0.031s
```





#### CREATE



#### ALTER



#### DROP



#### TRUNCATE

##### truncate、delete与drop区别？

**相同点：**

1. `truncate`和不带`where`子句的`delete`、以及`drop`都会删除表内的数据。
2. `drop`、`truncate`都是`DDL`语句（数据定义语言），执行后会自动提交。



**不同点：**

1. truncate 和 delete 只删除数据不删除表的结构；drop 语句将删除表的结构被依赖的约束、触发器、索引；
2. 一般来说，执行速度: drop > truncate > delete。



#### COMMENT



#### RENAME

# DML
## 概念

DML（Data Manipulation Language）数据操纵语言statements are used for managing data within schema objects.

由DBMS提供，用于让用户或程序员使用，实现对数据库中数据的操作。
DML分成交互型DML和嵌入型DML两类。
依据语言的级别，DML又可分成过程性DML和非过程性DML两种。
需要commit.



## 语法

#### SELECT



#### INSERT

#### UPDATE

#### DELETE

#### MERGE

#### CALL

#### EXPLAIN PLAN

#### LOCK TABLE



#### EXIST

#### IN

##### EXIST和IN的区别？

`exists`用于对外表记录做筛选。`exists`会遍历外表，将外查询表的每一行，代入内查询进行判断。当`exists`里的条件语句能够返回记录行时，条件就为真，返回外表当前记录。反之如果`exists`里的条件语句不能返回记录行，条件为假，则外表当前记录被丢弃。

`in`是先把后边的语句查出来放到临时表中，然后遍历临时表，将临时表的每一行，代入外查询去查找。

**子查询的表比较大的时候**，使用`exists`可以有效减少总的循环次数来提升速度；**当外查询的表比较大的时候**，使用`in`可以有效减少对外查询表循环遍历来提升速度。



#### WHERE



#### HAVING

##### having和where的区别？

- 二者作用的对象不同，`where`子句作用于表和视图，`having`作用于组。
- `where`在数据分组前进行过滤，`having`在数据分组后进行过滤。




# TCL
## 概念

TCL（Transaction Control Language）事务控制语言。



## 语法

### SAVEPOINT 设置保存点

### ROLLBACK  回滚

### SET TRANSACTION



### 查看和设置

```mysql
# 查看隔离级别
select @@transaction_isolation;
# 设置隔离级别
set session transaction isolation level read uncommitted;
set global transaction isolation level repeatable read;
```



# 其他

## SHOW

### 查看表结构 

```mysql
SHOW CREATE TABLE `it_blog`;
```

