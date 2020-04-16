# 查询数据库版本

Oracle:
```sql
select * from v$version;
-- Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
-- PL/SQL Release 12.2.0.1.0 - Production
-- CORE	12.2.0.1.0	Production
-- TNS for Linux: Version 12.2.0.1.0 - Production
-- NLSRTL Version 12.2.0.1.0 - Production
```

MySQL:
```sql
select version();
-- 8.0.19
```