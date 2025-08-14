`base` 目录是 PostgreSQL 的核心数据目录，包含所有数据库的表、索引等。

### ✅ 1. 分析哪些数据库或表最占空间

进入 PostgreSQL 命令行：

```bash
docker exec -it your-postgres-container psql -U your_user -d your_db
```

然后运行以下 SQL：

```sql
-- 查看各数据库大小
SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database ORDER BY pg_database_size(datname) DESC;

-- 查看最大的表
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 10;
```

### ✅ 2. 常见优化手段

#### （1）清理无用数据

- 删除不再需要的表、历史数据、日志表。
- 使用 `DELETE` 或 `TRUNCATE`。

#### （2）重建膨胀表（VACUUM FULL）

PostgreSQL 的 `UPDATE`/`DELETE` 会产生“膨胀”（bloat），占用额外空间。

```sql
-- 先查看膨胀情况（可选）
SELECT schemaname, tablename, n_dead_tup, pg_size_pretty(pg_table_size(schemaname || '.' || tablename)) AS size
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000;

-- 重建表（会锁表，建议低峰期）
VACUUM FULL VERBOSE table_name;
```

> ⚠️ `VACUUM FULL` 会锁表，且需要额外空间，建议在维护窗口执行。

#### （3）启用自动清理（Autovacuum）

确保 `postgresql.conf` 中启用：

```bash
autovacuum = on
log_autovacuum_min_duration = 0  # 记录自动清理日志，便于监控
```

