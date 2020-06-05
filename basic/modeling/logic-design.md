# 逻辑设计

## 范式化 VS 反范式化

### 01 宽表模式

- 问题
    - 数据冗余：相同的数据在一个表中出现了多次
    - 数据更新异常：修改一行中某列的值时，同时修改了多行数据
    - 数据插入异常：部分数据由于确实主键信息而无法写入表中
    - 数据删除异常：删除某一数据时不得不删除另一数据

- 应用场景
    - 配合列存储的数据报表应用

### 02 设计范式

- 第一范式：表中的所有字段都是不可再分的
- 第二范式：表中必须存在业务主键，并且非主键依赖于全部业务主键
- 第三范式：表中的非主键列之间不能相互依赖

### 03 范式化的优缺点

- 优点
    - 减少数据冗余
    - 减少数据的插入、更新、删除异常
    - 让数据之间的关系更加清晰

- 缺点
    - 查询需要关联多个表
    - 对查询性能有一定的影响，更难进行索引优化

## 04 反范式化的优缺点

- 优点
    - 可以减少表的关联
    - 可以更好的进行索引优化

- 缺点
    - 存在数据冗余及数据维护异常
    - 对数据的修改需要更多的成本