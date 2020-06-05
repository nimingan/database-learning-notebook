# MySQL： CTE 及窗口函数

## CTE

- 公共表表达式CTE(Common Table Expressions)
- 8.0以后的版本才可以使用

### 特性

- CTE生成一个命名零时表，并且只在查询期间有效
- CTE临时表在查询中可以多次引用和自应用

### 语法

```sql
WITH [RECURSIVE] 
cte_name [(column_list)] AS (
	query
)
[, cte_name [(column_list)] AS (
	query
)]
SELECT * FROM cte_name;
```

## 窗口函数

### 语法

```sql
function_name([exp])
OVER(
	[PARTITION BY exp [, ...]]
	[ORDER BY exp [ASC|DESC] [, ...]]
)
```

- 聚合函数
	- 聚合函数都可以作为窗口函数使用
- `ROW_NUMBER()`
	- 返回窗口分区内数据的行号
- `RANK()`
	- 类似于`row_number`，只是对于相同数据会产生重复的行号
- `DENSE_RANK()`
	- 类似于rank区别在于当组内某行数据重复时，虽然行号会重复，但后续的行号不会产生间隔。