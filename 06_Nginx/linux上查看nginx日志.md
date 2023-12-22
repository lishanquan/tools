linux系统下的nginx日志一般***默认***存储在***<u>/var/log/nginx</u>***目录下。

### 1.查看访问日志：

```shell
tail -f /var/log/nginx/access.log
```

### 2.查询错误日志：

```shell
tail -f /var/log/nginx/error.log
```

### 3.查看其他日志：

```shell
tail -f /var/log/nginx/other.log
```

