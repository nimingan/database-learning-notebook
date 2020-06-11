# MySQL : 存储引擎

* MySQL体系结构
    * 客户端
    * MySQL服务层
    * 存储引擎层

## 存储引擎

* `MyISAM`
    * 5.5之前版本默认存储引擎
    * 表由`MYD`和`MYI`组成
    * 特性
        * 并发性与锁级别 -> 表级锁
        * 表损坏修复
            * `check table tablename`
            * `repair table tablename`
        * MyISAM表支持的索引类型
            * 全文索引
            * blob前缀索引
        * MyISAM表支持数据压缩
    * 限制
        * 版本小于5.0默认表大小为4G,如存储大表则要修改`MAX_Rows`和`AVG_ROW_LENGTH`
    * 适用场景
        * 非事务型应用
        * 只读类应用
        * 空间类应用
* `Innodb`
    * 5.5及之后版本默认存储引擎
    * 使用表空间进行数据存储
        * `show variables like 'innodb_file_per_table'`
        * `innodb_file_per_table`
            * `ON`：独立表空间 `tablename.ibd`
                * 独立表空间可以通过`optimize table`命令收缩系统文件
                * 可以同时向多个文件刷新数据
            * `OFF`：系统表空间 `ibdataX`
                * 系统表空间无法简单的收缩文件大小
                * 系统表空间会产生IO瓶颈
                * `set global innodb_file_per_table = off`
                * 系统表空间切换为独立表空间
                    * 使用mysqldump导出所有数据库表数据
                    * 停止MySQL服务，修改参数，并删除Innodb相关文件
                    * 重启MySQL服务，重建Innodb系统表空间
                    * 重新导入数据
    * 特性
        * Innodb数据字典信息
        * Undo回滚段
        * 事务性存储引擎
            * 支持ACID
            * `Redo Log`
            * `Undo Log`
        * 支持行级锁
            * 什么是锁
                * 管理共享资源的并发访问
                * 用于实现事务的隔离性
            * 锁的类型
                * 共享锁（读锁）
                * 独占锁（写锁）
            * 锁的粒度
                * 表级锁
                    * mysql服务还是会对表进行表级锁，比如`alter table`
                * 行级锁
                    * 由存储引擎层实现
        * 状态检查
            * `show engine innodb status`
    * 适用场景
        * 适合于大多数OLTP应用
* `csv`
    * 特点
        * 数据以`csv`格式存储在文件中
            * `.csv` 表内容
            * `.csm` 表的原数据如表状态和数据量
            * `.frm` 表结构信息
        * 所有列必须都是不能为`NULL`的
        * 不支持索引
            * 不适合大表，不适合在线处理
        * 可以对数据文件直接编辑
* `Archive`
    * 特点
        * 以`zlib`对表数据进行压缩，磁盘I/O更少
        * 数据存储在`ARZ`为后缀的文件中
        * 只支持`insert`和`select`操作
        * 只允许在自增ID列上加索引
    * 使用场景
        * 日志和数据采集类应用
* `Memory`
    * 特点
        * 也成为`HEAP`存储引擎，所以数据保存在内存中
        * 支持`HASH`索引和`BTree`索引
            * 默认`HASH` -> 等值查找
            * `Btree` -> 范围查找
        * 所有字段都为固定长度
        * 使用表级锁
        * 最大大小由`max_heap_table_size`参数决定
    * 对比临时表
        * 临时表
            * 系统使用临时表
                * 超过限制使用`Myisam`临时表
                * 未超限制使用`Memory`表
            * `create temporary table`建立的临时表
    * 使用场景
        * 用于查找或者是映射表
        * 用于保存数据分析中产生的中间表
        * 用于缓存周期性聚合数据的结果表
        * Memory数据已丢失，所以要求数据可再生
* `Federated`
    * 特点
        * 提供了访问远程MySQL服务器上表的方法
        * 本地不存储数据，数据全部放在远程服务器上
        * 本地需要保存表结构和远程服务器的连接信息
    * 如何使用
        * 默认禁止，弃用需要在启动时增加federated参数