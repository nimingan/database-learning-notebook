# MySQL : 事务和高并发

## 事务
* 概念
    * 数据库执行操作的最小逻辑单元
    * 事务可以有一个SQL组成也可以由多个SQL组成
    * 组成事务的SQL要么全执行成功要么全执行失败

* 语法
    * `START TRANSACTION / BEGIN ... COMMIT/ROLLBACK`
* ACID特性
    * 原子性 A : 一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。
    * 一致性 C : 在事务开始之前和事务结束之后，数据库的完整性没有被破坏。
    * 隔离性 I : 要求每个读写事务的对象与其他事务的操作对象能相互分离，即该事务提交前对其他事务都不可见
    * 持久性 : 事务一旦提交了，其结果就是永久性的，就算发生了宕机等事故，数据库也能将数据恢复。

## 高并发
* 带来的问题
	* 脏读 ： 一个事务读取了另一个事务未提交的数据
	* 不可重复读 ： 一个事务前后两次读取的同一数据不一致
	* 幻读 ： 一个事务两次查询的结果集记录数不一致

## 事务隔离级别
* Innodb的隔离级别

| 隔离级别 | 脏读 | 不可重复度 | 幻读 | 隔离性 | 并发性 |
| --- | --- | --- | --- | --- | --- |
| 顺序读 `SERIALIZABLE` | N | N | N | 最高 | 最低 |
| 不可重复度 `REPEATABLE READ` | N | N | N | | aaa |
| 读已提交 `READ COMMITTED` | N | Y | Y | | aaa |
| 读未提交 `READ UNCOMMITTED` | Y | Y | Y | 最低 | 最高 |

* 设置事务的隔离级别
```
SET [PERSIST|GLOBAL|SESSION]
TRANSACTION ISOLATION LEVEL
{
    READ UNCOMMITTED
    | READ COMMITTED
    | REPEATABLE READ
    | SERIALIZABLE
}
```

## 阻塞和死锁

### 阻塞

* 由于不同锁之间的兼容关系，造成的A事务需要等待B事务释放其所占用的资源的现象

* 发现阻塞

```sql
select waiting_pid AS '被阻塞的线程',
waiting_query AS '被阻塞的SQL',
blocking_pid AS '阻塞线程',
blocking_query AS '阻塞SQL',
wait_age AS '阻塞时间',
sql_kill_blocking_query AS '建议操作'
FROM sys.innodb_lock_waits
WHERE (unix_timestamp()-unix_timestamp(wait_started))>30;
```

* 处理阻塞
    * 终止占用资源的事务
    * 优化占用资源事务的SQL，使其尽快释放资源

### 锁
* Innodb中的锁
    * 查询需要对资源加共享锁(S)
    * 数据修改需要对资源加排他锁(X)
    
* 死锁 ： 并行执行的多个事务相互之间占用了对方所需要的资源

* 发现死锁 `set global innodb_print_all_deadlocks = on;`
* 处理死锁
    * 数据库自行回滚占用资源少的事务(治标不治本)
    * 并发事务按照相同顺序占用资源