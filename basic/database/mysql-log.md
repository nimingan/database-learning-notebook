# MySQL : 日志及复制
* mysql服务层日志
    * 二进制日志
    * 慢查询日志
    * 通用日志
* mysql存储引擎层日志
    * innodb重做日志
    * innodb回滚日志

## 二进制日志
* 记录了所有对mysql数据库的修改事件，包括增删改查事件和对表结构的修改事件
* 格式
    * 基于段的格式 `binlog_format=STATEMENT`
        * 优点
            * 日志记录量相对较小，节约磁盘及网络I/O
        * 缺点
            * 必须记录上下文信息，保证语句在从服务器上执行结果和在主服务器上相同
                * 特点函数如`UUID()`,`user()`这样非确定性函数还是无法复制
            * 可能造成mysql复制的主备服务器数据不一致
    * 基于行的日志格式 `binlog_format=ROW`
        * 优点
            * 使mysql主从复制更加安全
                * 可以避免mysql复制中出现的主从不一致问题
            * 对每一行数据的修改比基于段的复制高效
            * 误操作而修改了数据库中的数据，同时又没有备份可以恢复时，我们就可以通过分析二进制日志，对日志中记录的数据修改操作做反向处理的方式来达到恢复数据的目的
        * 缺点
            * 记录日志量较大
                * `binlog_row_image=[FULL|MINIMAL|NOBLOB]`
    * 混合日志格式 `binlog_format=MIXED`
        * 特点
            * 根据SQL语句由系统决定在基于段和基于行的日志格式中进行选择
            * 数据量的大小由所执行的SQL语句决定
* 建议
    * `binlog_format=ROW`
    * `biglog_row_image=minimal`

## 复制
### 复制解决了什么问题
* 实现在不同服务器上的数据分布
    * 利用二进制日志增量进行
    * 不需要太多的带宽
        * 但是使用基于行的复制在进行大批量的更改时会对带宽带来一定的压力，特别是跨IDC环境下进行复制，应该分批进行
* 实现数据读取的负载均衡
    * 需要其他组件配合完成
        * 利用DNS轮询的方式把程序的读连接到不同的备份数据库适用`LVS`，`haproxy`这样的代理方式
    * 非共享架构，同样的数据分布在多台服务器上
* 增强了数据安全性
    * 利用备库的备份来减少主库负载
    * 复制并不能代替备份
* 实现数据库高可用和故障切换
* 实现数据库在线升级

### 二进制日志格式对复制的影响
* 基于SQL语句的复制`SBR`
    * 二进制日志是`statement`格式
    * 优点
        * 生成的日志量少，节约网络传输IO
        * 并不强制要求主从数据库的表定义完全相同
        * 相比于基于行的复制方式更为灵活
    * 缺点
        * 对于非确定性事件，无法保证主从复制数据的一致性
        * 对于存储过程，触发器，自定义函数进行修改也可能造成数据不一致
        * 相比于基于行的复制方式在从上执行时需要更多的行锁
* 基于行的复制`RBR`
    * 二进制日志是基于行的格式
    * 优点
        * 可以应用于任何SQL的复制包括非确定函数，存储过程等
        * 可以减少数据库锁的使用
    * 缺点
        * 要求主从数据库的表结构相同，否则可能会中断复制
        * 无法在从上单独执行触发器
* 混合模式
    * 根据实际内容在以上两者间切换

### 复制工作模式
1. 主将变更写入二进制日志
1. 从读取主的二进制日志变更并写入到relay_log中
1. 在从上重放relay_log中的日志
    * 基于SQL段的日志是在从库上重新执行记录的sql
    * 基于行的日志则是在从库上直接应用对数据库行的修改