# 什么影响了MySQL性能

## 硬件层面
- ?

## 软件层面
* 服务器系统
    * CentOS系统参数优化
        * 内核相关参数`/etc/sysctl.conf`
        * 增加资源限制`/etc/security/limit.conf`
        * 硬盘调度策略`/sys/block/devname/queue/scheduler`
    * 文件系统
        * Linux
            * EXT3
            * EXT4
            * XFS（ok）
* 数据库存储引擎的选择
    * MyISAM：不支持事务，表级锁
    * InnoDB：事务级存储引擎，完美支持行级锁，事务ACID特性
* 数据库参数配置
* 数据库结构设计和SQL语句