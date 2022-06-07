# 执行计划
## 为什么要关注执行计划？
- 了解 SQL 如何访问表中的数据，是使用全表扫描、还是索引等方式来获取的
- 了解 SQL 如何使用表中的索引，是否使用到了正确的索引
- 了解 SQL 锁使用的查询类型，是否使用到了子查询、关联查询等信息



## explain关键字

可通过 EXPLAIN 来获取到 SQL的执行计划

| id   | select_type | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :--------- | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | table_name | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 100  | 33.33    | Using where |



## 执行计划表

### id列

id 会有两种值：

- 数值：查询中的 SQL 数据对数据库对象操作的顺序
	- 当 ID 相同时由上到下执行。
	- 当 ID 不同时，由大到小执行

- NULL：这一行数据是由另外两个查询进行 union 后产生的



#### 数值

##### id数值相同时

```mysql
EXPLAIN SELECT * FROM `it_blog` ib INNER JOIN `it_blog_classify` ibc ON ib.`classify_id` = ibc.`id`
```



执行计划如下：

| id   | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                            | rows | filtered | Extra |
| :--- | ----------- | ----- | ---------- | ------ | ------------- | ------- | ------- | ------------------------------ | ---- | -------- | ----- |
| 1    | SIMPLE      | ib    | NULL       | ALL    | idx_classify  | NULL    | NULL    | NULL                           | 1699 | 100.00   | Null  |
| 1    | SIMPLE      | ibc   | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | l-sixth-service.ib.classify_id | 1    | 100.00   | Null  |



此时SQL执行循序是从上到下执行，先执行`it_blog`表的查询，再执行`it_blog_classify`表的查询。



##### id数值不同时

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



此时SQL执行顺序是由大到小执行，先执行`it_blog_classify`表子查询，再执行外面的`it_blog  `表查询。



#### NULL

```mysql
EXPLAIN SELECT `id`, `title` FROM `it_blog` WHERE `id` = 10
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 20
```



执行计划如下：

| id   | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
| :--- | ------------ | ---------- | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | --------------- |
| 1    | PRIMARY      | it_blog    | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| 2    | UNION        | it_blog    | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL | NULL     | Using temporary |



此时执行计划结果中id为NULL的执行计划表示这一行数据是由另外两个查询进行 union 后产生的。



### select_type列

有以下几种类型：

- SIMPLE：不包含子查询或是 UNION 操作的查询
- PRIMARY：查询中如果包含任何子查询，那么最外层的查询则被标记为 PRIMARY
- SUBQUERY：SEL ECT 列表中的子查询
- DEPENDENT SUBQUERY：依赖外部结果的子查询
- UNION：union 操作的第二个或是之后的查询的值为 union
- UNION RESULT：UNION 产生的结果集
- DEPENDENT UNION：当 UNION 作为子查询时，第二或是第二个后的查询的 select_type 值
- DERIVED：出现在 FROM 子句中的子查询
- MATERIALIZED：



#### SIMPLE

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `id` = 10
```



执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL  |



`select_type` 值为 `SIMPLE` 时表示该查询是简单查询，不包括任何子查询或UNION查询。



#### PRIMARY & SUBQUERY

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



当一个查询语句中包括子查询时，`PRIMARY` 表示该表的查询时子查询的外层查询，`SUBQUERY` 则表示该表是子查询。



#### DEPENDENT SUBQUERY



#### UNION & UNION RESULT

```mysql
EXPLAIN SELECT `id`, `title` FROM `it_blog` WHERE `id` IN (10, 30)
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 20
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 40
```



执行计划如下：

| id   | select_type  | table        | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
| :--- | ------------ | ------------ | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | --------------- |
| 1    | PRIMARY      | it_blog      | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL  | 2    | 100.00   | Using where     |
| 2    | UNION        | it_blog      | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| 3    | UNION        | it_blog      | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| NULL | UNION RESULT | <union1,2,3> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL | NULL     | Using temporary |



`UNION` 表示该表的查询操作是联合查询的第二个或是之后的查询；`UNION RESULT` 则表示是联合查询产生的结果集。



#### DEPENDENT UNION



#### DERIVED

```mysql
EXPLAIN SELECT drived_table.* FROM (
	SELECT COUNT(`id`), `classify_id` FROM `it_blog` GROUP BY `classify_id`) drived_table 
WHERE drived_table.`classify_id` = 1
```



执行计划如下：

| id   | select_type | table      | partitions | type  | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ---------- | ---------- | ----- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | PRIMARY     | <derived2> | NULL       | ref   | <auto_key0>   | <auto_key0>  | 8       | const | 10   | 100.00   | NULL        |
| 2    | DERIVED     | it_blog    | NULL       | index | idx_classify  | idx_classify | 8       | NULL  | 1718 | 100.00   | Using index |



`DERIVED` 表示出现在 `FROM` 子句中的子查询。



#### MATERIALIZED

```mysql
EXPLAIN SELECT * FROM `it_blog_classify` WHERE `id` IN (
	SELECT `classify_id` FROM `it_blog` WHERE `classify_id` NOT IN (
		SELECT `id` FROM `it_blog_classify`))
```



执行计划如下：

| id   | select_type  | table            | partitions | type   | possible_keys | key          | key_len | ref                                 | rows | filtered | Extra                    |
| :--- | ------------ | ---------------- | ---------- | ------ | ------------- | ------------ | ------- | ----------------------------------- | ---- | -------- | ------------------------ |
| 1    | PRIMARY      | it_blog_classify | NULL       | ALL    | PRIMARY       | NULL         | NULL    | NULL                                | 10   | 100.00   | Using where              |
| 1    | PRIMARY      | <subquery2>      | NULL       | eq_ref | <auto_key>    | <auto_key>   | 8       | l-sixth-service.it_blog_classify.id | 1    | 100.00   | NULL                     |
| 2    | MATERIALIZED | it_blog          | NULL       | index  | idx_classify  | idx_classify | 8       | NULL                                | 1718 | 100.00   | Using where; Using index |
| 3    | SUBQUERY     | it_blog_classify | NULL       | index  | PRIMARY       | idx_user     | 8       | NULL                                | 10   | 100.00   | Using index              |



### table列

表示执行计划表中的数据是由哪个表输出的

- 表名：指明从哪个表中获取数据，有原始表名，或则别名
- `<unionM,N>`：表示由 ID 为 M.N 查询 union 产生的结果集
- `<derived N>/<subquery N>`：由 ID 为 N 的查询产生的结果集



### partitions列

只有在查询分区表的时候，才会显示分区表的 ID



### type列

通常通过 type 可以知道查询使用的连接类型。



>在 MySQL 中，并不是只有通过 join 产生的关联查询才叫关联查询。
>
>就算只查询一个表，也会认为是一个连接查询



type 按照性能从高到低排列如下：

- `system` （性能最高）

	这是 const 连接类型的一个特列，当查询的表只有一行时使用。

- `const`

	表中有且只有一个匹配的行时使用，如对住建或是唯一索引的查询，这是效率最高的连接方式。

- `eq_ref`

	唯一索引或主键引查找，对于每个索引键，表中只有一条记录与之匹配；是以前面表返回的数据行为基础，对于每一行数据都会从本表中读取一行数据。

- `ref`

	非唯一索引查找、返回匹配某个单独值的所有行。

- `ref_or_null`

	类似于 ref 类型的查询，但是附加了对 NULL 值列的查询。

- `index_merge`

	该连接类型表示使用了索引合并优化方法；mysql 5.6 之前，一般只支持一个索引查询，后来支持索引合并之后，可以支持多个索引查询。

- `range`

	索引范围扫描，常见于 between、`>`、`<` 这样的查询条件。

- `index`

	FULL index Scan 全索引扫描，同 ALL 的区别是，遍历的是索引树。

- `ALL` （性能最底）

	FULL TABLE Scan 权标扫表这是效率最差的连接方式。

- `NULL`

	 MySQL不访问任何表或索引，直接返回结果。



#### system



#### const

#### eq_ref



#### ref

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` = 2
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify  | idx_classify | 8       | const | 3    | 100.00   | NULL  |



`ref` 表示非唯一索引查找、返回匹配某个单独值的所有行。



#### ref_or_null



#### index_merge

```mysql
SELECT * FROM `it_blog` WHERE `user_id` = 1 OR `classify_id` = 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type        | possible_keys                            | key                                      | key_len | ref  | rows | filtered | Extra                                                        |
| :--- | ----------- | ------- | ---------- | ----------- | ---------------------------------------- | ---------------------------------------- | ------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| 1    | SIMPLE      | it_blog | NULL       | index_merge | idx_user_tag_classify,idx_classify_title | idx_user_tag_classify,idx_classify_title | 8,8     | NULL | 6    | 100.00   | Using sort_union(idx_user_tag_classify,idx_classify_title); Using where |



#### range

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` IN (1, 2, 3, 4, 5, 6)
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------------ | ------- | ---- | ---- | -------- | --------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | idx_classify  | idx_classify | 8       | NULL | 9    | 100.00   | Using index condition |



`range` 表示索引范围扫描，常见于 between、`>`、`<` 和 `IN` 查询这样的查询条件。



#### index



#### ALL

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `title` = 'Java'
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1725 | 10.00    | Using where |



`ALL` 表示全表扫描，是效率最低的方式。



#### NULL

```mysql
EXPLAIN SELECT 1
```



执行计划如下：



| id   | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
| :--- | ----------- | ----- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | -------------- |
| 1    | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | No tables used |



此时type列的值为NULL，表示没有访问任何表或索引，直接返回结果。



### possible_keys列

指出查询中可能会用到的索引，优化器会把这里边的索引都试一遍，然后选一个开销最小的，如果都不太行，那就直接全表扫描。



> 如果表中建立的索引过多，执行SQL时，优化器就会花费更多的时间在选择索引上。因此一般来说表中的索引个数不要超过5个。



### key列

possible_keys 列出的是可能使用到的索引，key 则是表示实际使用到的索引

- 如果为 NULL，则表示该表中没有使用到索引
- 如果出现的值，没有存在 possible_keys 中，查询可能使用到的是覆盖索引





### key_len列

实际使用索引的最大长度。

**注意：**比如在一个联合索引中，如果有 3 列且总字节长度是 100 字节，key_len 可能少于 100 字节的，比如只有 30 个字节，这就说明在查询中没有到联合索引中的所有列，而是只使用到了联合索引中的部分列。

**注意：** 这一列的字节计算方式是以表中定义列的字节方式，而不是数据的实际长度



### ref列

指出哪些列或常量被用于索引查找；





### rows列

有两方面的含义：

- 根据统计信息预估的扫描的行数

- 在关联查询中，也表示内嵌循环的次数

	每获取一个值，就要对目标表进行一次查找，循环越多，性能就越差



```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



### filtered列

与 rows 有一定的关系，表示返回结果的行数占需要读取行数（rows）的百分比。也是一个预估值，不太准确，但是可以在一定程度上可以预估 mysql 的查询成本。该值越高，效率也越高。



```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `title` = 'Java' AND `classify_id` = 1
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify  | idx_classify | 8       | const | 2    | 10.00    | Using where |



### Extra列

包含了不适合在其他列中显示的一些额外信息。常见的值有如下：

| 值                                 | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| NULL                               |                                                              |
| Distinct                           | 优化 distinct 操作，在找到第一匹配的元组后即停止找同样值的动作 |
| Not exists                         | 使用 not exists 来优化查询                                   |
| Using filesort                     | 使用文件来进行排序，通常会出现在 order by 或 group by 查询中 |
| Using index                        | 使用了覆盖索引进行查询；<br/>覆盖索引：查询中使用到的信息是完全可以通过索引信息来获取的，而不用去从表中的数据进行访问 |
| Using index condition; Using where |                                                              |
| Using where                        | 需要在 MySQL 服务器层使用 Where 条件来过滤数据               |
| Using where; Using index           |                                                              |
|                                    |                                                              |
| Using temporary                    | MySQL 需要使用临时表来处理查询，常见于排序、子查询、和分组查询 |
| select tables optimizedaway        | 直接通过索引来获得数据，不用访问标                           |



#### NULL

```mysql
EXPLAIN SELECT * FROM `it_blog` 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1779 | 100.00   | NULL  |



#### Distinct





#### Not exists  





#### Using filesort





#### Using index

```mysql
EXPLAIN SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys                      | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ---------------------------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify,idx_user_tag_classify | idx_classify | 8       | const | 2    | 100.00   | Using index |



`Using index` 表示where 和select中需要的字段都能够直接通过一个索引字段获取，无需再回表查询，当查询涉及的列都是某一单独索引的组成部分时即为此种情况，这实际上就是索引类型中覆盖索引。

上述查询的SQL条件和结果列都在 `idx_classify` 索引树中，因此无需再回表查询。



#### Using index condition; Using where





#### Using where

- 非索引where匹配查询

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `read_status` = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1779 | 100.00   | Using where |



`Using where` 表示where条件中的查询没有使用到索引来限制要返回的select结果，执行计划中 `type` 是ALL类型。



#### Using where; Using index

```mysql
EXPLAIN SELECT `classify_id` FROM `it_blog` WHERE `classify_id`%2 = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                    |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------------ | ------- | ---- | ---- | -------- | ------------------------ |
| 1    | SIMPLE      | it_blog | NULL       | index | NULL          | idx_classify | 8       | NULL | 1779 | 100.00   | Using where; Using index |



`Using where; Using index` 表示sql使用了覆盖索引，所有字段均从索引中获取，同时在从索引中查询出初步结果后，还需要使用组成索引的部分字段进一步进行条件筛选，而不是说需要回表获取完整行数据。



#### 	Using temporary





> 当该列的值出现了 Using temporary 时，就需要重点关注了，通常来说性能不会太好





#### select tables optimizedaway



