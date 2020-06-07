# MySQL : 索引及sql优化

## SQL优化的方向

- 优化SQL查询所涉及到的表中的索引
- 改写SQL以达到更好的利用索引的目的

## 索引

### 索引的作用
* 告诉存储引擎如何快速的查找到所需要的数据
* 索引大大减少了存储引擎需要扫描的数据量
* 索引可以帮助我们进行排序以避免使用临时表
* 索引可以把随机IO转换成顺序IO

### Innodb支持的索引类型
* Btree索引：以B+树的结构存储索引数据
* 自适应Hash索引
    * 基于Hash表实现，只有查询条件精确匹配Hash索引中的所有列时，才能够使用到hash索引
    * 对于Hash索引中的所有列，存储引擎都会为每一行计算一个Hash码，Hash索引中存储的就是Hash码
* 全文索引
* 空间索引

### Btree索引
* 特点：
    * 适用于全值匹配的查询
        * `class_name = 'mysql'`
        * `class_name in ('mysql','oracle')`
    * 适用于处理范围查找
        * `between ... and ....`
        * `>`,`<`
    * 匹配列前缀查询
        * `order_sn like '9876%'`
    * 从索引的最左侧列开始查找列
        * `create index idx_title_study on t_course(title,study)`
        * `study_cnt>3000 and title='Mysql'`
    * 只访问索引的查询
* 限制
	* 如果不是按照索引最左列开始查找，则无法使用索引
	* 使用索引时不能调过索引中的列
	* `NOT IN`和`<>`操作无法使用索引
	* 索引列上不能使用表达式或是函数
	* 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引

### hash索引
* 限制
	* Hash索引必须进行二次查找
	* Hash索引无法用于排序
	* Hash索引不支持部分索引查找也不支持范围查找
	* Hash索引中Hash码的计算可能存在Hash冲突

### 索引使用的误区
* 索引越多越好？NO
    * 索引会增加写操作的成本
    * 太多的索引会增加查询优化器的选择时间
* 使用`IN`列表查询不能用到索引？NO
    * mysql会根据`in`列表进行分析
* 查询过滤顺序必须和索引键顺序相同才可以使用到索引？NO

### 索引优化策略
* 索引列上不能使用表达式或函数
    * `select ... from product where to_days(out_date) - to_days(current_date) <=30`NO  :-1:
    * `select --- from product where out_date <= date_add(current_date,interval 30 day)`YES  :+1:
* 前缀索引和索隐裂的选择性
    * `CREATE INDEX index_name ON table(col_name(n))`;
    * **索引的选择性** : 不重复的索引值和表的记录数的比值
* 联合索引
    * 选择性高的列放在联合索引的最左侧
    * 使用最频繁的列放在联合索引的最左侧
    * 尽量把字段长度小的列放在联合索引列的最左侧 -> 一页中存的索引多
* 覆盖索引
    * 可以优化缓存，减少磁盘IO操作
    * 可以减少随机IO，变随机IO操作为顺序IO操作
    * 可以避免对Innodb主键索引的二次查询
    * 可以避免MyISAM表进行系统调用
    * 无法使用的情况
        * 存储引擎不支持
        * 查询中使用了太多的列
        * 使用了双%号的like查询
* 利用索引优化锁
    * 索引可以减少锁定的行数
    * 索引可以加快处理速度，也加快了锁的释放
* 删除重复和冗余的索引
    * `pk(id) uk(id) index(id)` 重复
    * `index(a) index(a,b)` 冗余
    * `pk(id) index(a,id)` 冗余
    * `pt-duplicate-key-checker h=127.0.0.1` 冗余检测
* 查找未被使用过的索引
* 更新索引统计信息及减少索引碎片
    * `analyze table table_name`
    * `optimize table table_name`
        * 使用不当会导致锁表

### 在哪些列上建立索引
* WHERE子句中的列
    * 可以考虑对筛选率高的列建立索引，提高查询效率
* 包含在`ORDER BY`、`GROUP BY`、`DISTINCT`中的字段
* 多表`JOIN`的关键列

## SQL优化
* 使用outer join代替not in
* 使用CTE代替子查询
* 拆分复杂的大SQL为多个简单的小SQL
* 巧用计算列优化查询