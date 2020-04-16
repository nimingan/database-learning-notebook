# 查询数据库版本

oracle:
```sql
select * from v$version;
-- Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
```

mysql:
```sql
select version();
-- 8.0.19
```