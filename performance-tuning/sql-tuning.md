# SQL优化
## 数据库优化工具
### 自动化工具
- Oracle
    - Automatic Database Diagnostic Monitor (ADDM)
    - SQL Tuning Advisor
    - SQL Access Advisor
    - SQL Plan Management
    - SQL Performance Analyzer
- MySQL
    - [占坑]

### 手动工具
- Oracle
    - Execution Plans
    - Real-Time SQL Monitoring and Real-Time Database Operations
    - Application Tracing
    - Optimizer Hints
- MySQL
    - Execution Plans

## 执行计划语法 - Execution Plan
### Oracle

### MySQL
- 语法：[EXPLAIN Statement](https://dev.mysql.com/doc/refman/8.0/en/explain.html)
```sql
EXPLAIN
    [explain_type]
    {explainable_stmt | FOR CONNECTION connection_id};

EXPLAIN ANALYZE select_statement;
```
- `EXPLAIN`可以用于`SELECT`、`DELETE`、`INSERT`、`REPLACE`和`UPDATE`表达式。
- `EXPLAIN`在`MySQL 8.0.19`和之后的版本，可以用于`TABLE`表达式。`EXPLAIN table_name`效果和`DESC table_name`效果相同。
- 举个栗子
```sql
# USE sakila;
EXPLAIN SELECT a.address_id, a.address, a.address2, a.city_id, a.location, c.city, c.country_id
        FROM address a LEFT JOIN city c ON a.city_id = c.city_id
        WHERE a.city_id = 300;
# +----+-------------+-------+------------+-------+----------------+----------------+---------+-------+------+----------+-------+
# | id | select_type | table | partitions | type  | possible_keys  | key            | key_len | ref   | rows | filtered | Extra |
# +----+-------------+-------+------------+-------+----------------+----------------+---------+-------+------+----------+-------+
# |  1 | SIMPLE      | a     | NULL       | ref   | idx_fk_city_id | idx_fk_city_id | 2       | const |    2 |   100.00 | NULL  |
# |  1 | SIMPLE      | c     | NULL       | const | PRIMARY        | PRIMARY        | 2       | const |    1 |   100.00 | NULL  |
# +----+-------------+-------+------------+-------+----------------+----------------+---------+-------+------+----------+-------+

EXPLAIN address;
# +-------------+-------------------+------+-----+-------------------+-----------------------------------------------+
# | Field       | Type              | Null | Key | Default           | Extra                                         |
# +-------------+-------------------+------+-----+-------------------+-----------------------------------------------+
# | address_id  | smallint unsigned | NO   | PRI | NULL              | auto_increment                                |
# | address     | varchar(50)       | NO   |     | NULL              |                                               |
# | address2    | varchar(50)       | YES  |     | NULL              |                                               |
# | district    | varchar(20)       | NO   |     | NULL              |                                               |
# | city_id     | smallint unsigned | NO   | MUL | NULL              |                                               |
# | postal_code | varchar(10)       | YES  |     | NULL              |                                               |
# | phone       | varchar(20)       | NO   |     | NULL              |                                               |
# | location    | geometry          | NO   | MUL | NULL              |                                               |
# | last_update | timestamp         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED on update CURRENT_TIMESTAMP |
# +-------------+-------------------+------+-----+-------------------+-----------------------------------------------+
```
- 执行计划结果说明[EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)
    - ID：ID表示查询执行的顺序；ID相同时由上到下执行；ID不同时，由大到小执行。
    - select type：
        - SIMPLE：不含子查询或是UNION操作的简单查询
        - PRIMARY：查询中如果包含任何子查询，那么最外层的查询则被标记为PRIMARY
        - SUBQUERY：SELECT类别中的第一个子查询
        - DEPENDENT SUBQUERY：依赖外部结果的子查询
        - UNION：union操作的第二个或是之后的查询的值为union
        - DEPENDENT UNION：当UNION作为子查询时，第二或是第二个后的查询的select_type值
        - UNION RESULT：UNION产生的结果集
        - DERIVED：出现在FROM子句中的子查询
    - table
        - 指明是从哪个表中获取数据
        - `<unionM,N>`由ID为M，N查询union产生的结果集
        - `<derived N>/<subquery N>`由ID为N的查询产生的结果
    - partitions
        - 对于分区表，显示查询的分区ID
        - 对于非分区表，显示NULL
    - type
        - system:这是const连接类型的一个特例，当查询的表只有一行时使用
        - const:表中有且只有一个匹配的行时使用，如对主键或是唯一索引的查询，这是效率最高的连接方式
        - eq_ref:唯一索引或主键索引查找，对于每个索引键，表中只有一条记录与之匹配
        - ref:非唯一索引查找，返回匹配某个单独值的所有行
        - ref_or_null:类似于ref类型的查询，但是附加了对NULL值列的查询
        - index_merge:该连接类型表示使用了索引合并优化方法
        - range:索引范围扫描，常见于between、>、<这样的查询条件
        - index:FULL index Scan全索引扫描，同ALL的区别是，遍历的是索引树
        - ALL:FULL table Scan全表扫描，这是效率最差的连接方式
    - possible keys:可能用到哪些索引
    - key:查询时实际用到的索引
    - key_len:实际使用索引的最大长度
    - ref:指出哪些列或常量被用于索引查找
    - rows:根据统计信息预估的扫描的行数
    - filtered:表示返回结果的行数占需读取行数的百分比
    - extra
        - Distinct:优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作
        - Not exists:使用not exists来优化查询
        - Using filesort:使用文件来进行排序，通常会出现在order by或group by查询中
        - Using index:使用了覆盖索引进行查询
        - Using temporary:MySQL需要使用临时表来处理查询，常见于排序，子查询和分组查询
        - Using where:需要MySQL服务器层使用WHERE条件来过滤数据
        - select tables optimized away:直接通过索引来获得数据，不用访问表

## 参考文档
- [Database 2 Day + Performance Tuning Guide](https://docs.oracle.com/database/121/TDPPT/toc.htm)
- []