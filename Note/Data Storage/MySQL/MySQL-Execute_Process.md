# 查询SQL

查询语句的执行流程如下：权限校验、查询缓存、分析器、优化器、权限校验、执行器、引擎。



举个例子，查询语句如下：

```mysql
select * from user where id > 1 and name = '大彬';
```



1. 首先检查权限，没有权限则返回错误；
2. MySQL8.0以前会查询缓存，缓存命中则直接返回，没有则执行下一步；
3. 词法分析和语法分析。提取表名、查询条件，检查语法是否有错误；
4. 两种执行方案，先查 `id > 1` 还是 `name = '大彬'`，优化器根据自己的优化算法选择执行效率最好的方案；
5. 校验权限，有权限就调用数据库引擎接口，返回引擎的执行结果。



## 存储引擎

### Innodb

根据主键查询时直接走主键索引，从主键索引的根结点开始遍历，然后查到叶子结点，直接得到行数据；非主键索引查询时，先从非主键索引树的叶子节点中获取到主键，再根据主键回表查询主键索引，得到行数据；全表扫描时直接遍历主键索引的叶子节点，得到行数据。



### MyISAM

MyISAM下的表数据都是单独存储一个文件进行存储的，并不像InnoDB一样，存储在主键索引中，所以MyISAM不管查询是走主键索引，还是辅助键索引，通通都要遍历到叶子结点，从叶子结点获得真实数据的地址，再通过地址找到真实数据。

# 更新SQL

更新语句执行流程如下：分析器、权限校验、执行器、引擎、`redo log`（`prepare`状态）、`binlog`、`redo log`（`commit`状态）



举个例子，更新语句如下：

```mysql
update user set name = '大彬' where id = 1;
```



1. 先查询到 id 为1的记录，有缓存会使用缓存。
2. 拿到查询结果，将 name 更新为大彬，然后调用引擎接口，写入更新数据，innodb 引擎将数据保存在内存中，同时记录`redo log`，此时`redo log`进入 `prepare`状态。
3. 执行器收到通知后记录`binlog`，然后调用引擎接口，提交`redo log`为`commit`状态。
4. 更新完成。

为什么记录完`redo log`，不直接提交，而是先进入`prepare`状态？

假设先写`redo log`直接提交，然后写`binlog`，写完`redo log`后，机器挂了，`binlog`日志没有被写入，那么机器重启后，这台机器会通过`redo log`恢复数据，但是这个时候`binlog`并没有记录该数据，后续进行机器备份的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。



# 相关概念

## 回表

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` = 1 
```

查询上述SQL时，先通过索引 `classif_id` 获取到叶子节点上的主键id，此时表中的其他数据还需要通过主键去查询。



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :---- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 100.00   | NULL  |



通过二级索引找到B+树中的叶子结点，但是二级索引的叶子节点的内容并不全，只有索引列的值和主键值。需要拿着主键值再去聚簇索引（主键索引）的叶子节点中去拿到完整的用户记录，这个过程叫做回表。

二级索引叶子节点中的主键id的排布就没有任何规律了，进行回表的时候，极有可能出现主键`id`所在的记录在聚簇索引叶子节点中反复横跳的情况，也就是随机IO。如果目标数据页恰好在内存中的话效果倒也不会太差，但如果不在内存中，还要从磁盘中加载一个数据页的内容（16KB）到内存中，这个速度就太慢了。

因此要尽量减少回表操作带来的损耗，总结起来就是两点：

1. 能不回表就不回；
2. 必须回表就减少回表的次数。



## 覆盖索引

如果辅助索引中已经包含了所有需要读取的列数据，就不需要回表，这种查询方式称为**覆盖索引**（或**索引覆盖**），如下SQL所示：

```mysql
SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` = 1 
```



此时索引页中就包含了所查询的所有数据，无需回表查询，执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys                            | key                | key_len | ref   | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :--------------------------------------- | :----------------- | :------ | :---- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_user_tag_classify,idx_classify_title | idx_classify_title | 9       | const | 2    | 100.00   | Using index |



覆盖索引在MySQL中，仅仅是针对InnoDB存储引擎而言的。准确的说，是针对聚簇索引和非聚簇索引共存的情况下才能起作用的。



## ICP索引下推

> To see how this optimization works, consider first how an index scan proceeds when Index Condition Pushdown is not used:
>
> 1. Get the next row, first by reading the index tuple, and then by using the index tuple to locate and read the full table row.
> 2. Test the part of the `WHERE` condition that applies to this table. Accept or reject the row based on the test result.
>
> When Index Condition Pushdown is used, the scan proceeds like this instead:
>
> 1. Get the next row's index tuple (but not the full table row).
> 2. Test the part of the `WHERE` condition that applies to this table and can be checked using only index columns. If the condition is not satisfied, proceed to the index tuple for the next row.
> 3. If the condition is satisfied, use the index tuple to locate and read the full table row.
> 4. Test the remaining part of the `WHERE` condition that applies to this table. Accept or reject the row based on the test result.



### ICP实践

现有如下联合索引：

| 索引名称           | 字段               | 索引类型 | 索引方法 |
| ------------------ | ------------------ | -------- | -------- |
| idx_classify_title | classify_id, title | NORMAL   | BTREE    |



执行如下SQL获取 `classify_id` 为1且 `title` 中包含'异常设计'的数据：

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` = 1 AND `title` LIKE '%异常设计%'
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra                 |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :-------------------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 11.11    | Using index condition |



由于联合索引的叶子节点的记录是先按照 `classify_id` 字段排序，`classify_id` 字段相同的情况下再按照 `title` 字段排序，因为把`%`加在 `title` 字段前面时无法利用索引的顺序性来进行快速比较的，也就是说这条查询语句中只有 `classify_id` 字段可以使用索引进行快速比较和过滤。正常情况下查询过程是这个样子的：

1. InnoDB使用联合索引查出所有 `classify_id` 为1的二级索引数据，得到主键值；
2. 拿到主键索引进行回表，到聚簇索引中拿到完整记录；
3. InnoDB把完整记录返回给MySQL的Server层，在Server层过滤出 `title` 包含'异常设计'的数据。



此时如果 `classify_id` 查询出来的结果较多，就需要将大量数据返回给Server层进行过滤。但是实际上是可以在InnoDB存储引擎中过滤出`title` 包含'异常设计'的数据主键的。这种在存储引擎中提前过滤数据的行为就称为索引下推（Index Condition Pushdown，**ICP**），或索引条件下推。

准确来说就是过滤的动作由下层的存储引擎层通过使用索引来完成，而不需要上推到Server层进行处理，使用ICP的方式能有效减少回表的次数。

ICP是在MySQL5.6之后完善的功能，是默认开启的，对于二级索引，只要能把条件甩给下面的存储引擎，存储引擎就会进行过滤，不需要进行干预。



> 即使满足索引下推的使用条件，查询优化器也未必会使用索引下推，因为可能存在更高效的方式。



### ICP设置

- 查看索引下推设置

```mysql
# 查看优化器开关
SHOW VARIABLES LIKE 'optimizer_switch';
```

​	

| Variable_name    | Value                                 |
| ---------------- | ------------------------------------- |
| optimizer_switch | ..., index_condition_pushdown=on, ... |



- 开启索引下推

```mysql
SET optimizer_switch="index_condition_pushdown=on";
```



- 关闭索引下推

```mysql
SET optimizer_switch="index_condition_pushdown=off";
```



关闭索引下推后再执行同样的SQL，其执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 11.11    | Using where |



和打开索引下推时的执行计划区别在于Extra信息发生变化，没有再使用索引条件。



