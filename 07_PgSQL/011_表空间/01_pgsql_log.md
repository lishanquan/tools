PostgreSQL 的 `log` 目录通常是 **最占空间但最容易清理** 的部分。

### ✅ 1. 确认日志是否可清理

bash深色版本

```bash
ls -lh ./log/
```

- 如果是 `.log` 文件，且日期较久（如几个月前），**可以安全删除或归档**。
- 这些是日志文件，不影响数据库运行。



### ✅ 2. 立即清理旧日志

bash深色版本

```bash
# 删除 30 天前的日志
find ./log/ -name "*.log" -mtime +30 -delete

# 或者只保留最近 7 天
find ./log/ -name "*.log" -mtime +7 -exec rm -f {} \;
```

> ⚠️ 建议先 `gzip` 压缩归档再删除，以防需要排查问题：
>
> ```bash
> find ./log/ -name "*.log" -mtime +7 -exec gzip {} \;
> ```



### ✅ 3. 配置 PostgreSQL 自动轮转和清理

编辑 PostgreSQL 配置文件 `postgresql.conf`（通常在 `/mnt/docker/pgsql/data/postgresql.conf`）：



```bash
# 日志设置
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d.log'
log_rotation_size = 100MB        # 每 100MB 轮转一次
log_rotation_age = 1d           # 每天轮转一次
log_retention_days = 7          # 保留 7 天（关键！）
```

> ✅ 设置 `log_retention_days` 后，PostgreSQL 会自动删除旧日志。