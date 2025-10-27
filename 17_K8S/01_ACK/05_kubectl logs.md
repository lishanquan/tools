`kubectl logs` 是 Kubernetes 中用于查看 Pod 中容器日志的核心命令。它可以帮助你排查应用问题、监控运行状态、分析错误信息。

------

## 📌 一、基本语法

```bash
kubectl logs [POD_NAME] [OPTIONS]
```

如果 Pod 中有多个容器，需要指定容器名。

------

## 🚀 二、常用命令与用法

### 1. 查看某个 Pod 的日志（单容器）

```bash
kubectl logs my-pod
```

> 假设 `my-pod` 只有一个容器。

------

### 2. 查看指定容器的日志（多容器 Pod）

```bash
kubectl logs my-pod -c container-name
```

- `-c` 或 `--container`：指定容器名称。
- 适用于一个 Pod 中有多个容器（如主应用 + sidecar）。

------

### 3. 实时查看日志（类似 `tail -f`）

```bash
kubectl logs -f my-pod
```

- `-f` 或 `--follow`：持续输出新日志，实时跟踪。

> 可结合容器使用：`kubectl logs -f my-pod -c app-container`

------

### 4. 查看最近 N 行日志

```bash
kubectl logs --tail=20 my-pod
```

- 只显示最后 20 行日志。
- 适用于日志太多时快速查看末尾内容。

------

### 5. 查看过去一段时间内的日志

```bash
kubectl logs --since=1h my-pod
```

- 查看过去 1 小时的日志。
- 支持单位：`s`（秒）、`m`（分钟）、`h`（小时）

**示例：**

```bash
kubectl logs --since=30m my-pod        # 最近30分钟
kubectl logs --since=2h --tail=50 my-pod # 2小时内，最多50行
```

------

### 6. 指定时间范围查看日志

```bash
kubectl logs --since-time="2025-01-01T00:00:00Z" my-pod
```

- 使用 RFC3339 格式。
- 注意时区：通常用 UTC 时间。

------

### 7. 查看之前容器的日志（重启/崩溃后）

```bash
kubectl logs my-pod --previous
```

- `-p` 或 `--previous`：查看上一个容器实例的日志。
- 对排查容器崩溃（CrashLoopBackOff）非常有用。

> 例如：容器启动失败，重启后原实例日志已丢失，用 `--previous` 可查看失败时的日志。

------

### 8. 查看指定命名空间的 Pod 日志

```bash
kubectl logs -n namespace-name my-pod
```

- `-n` 或 `--namespace`：指定命名空间。
- 默认是 `default` 命名空间。

------

### 9. 结合标签（Label）查看日志

```bash
kubectl logs -l app=nginx
```

- `-l` 或 `--selector`：根据标签选择 Pod。
- 如果匹配多个 Pod，会依次显示每个 Pod 的日志。

**只看其中一个（配合 `--max-log-requests`）：**

```bash
kubectl logs -l app=nginx --max-log-requests=1
```

------

### 10. 限制日志输出大小

```bash
kubectl logs my-pod --limit-bytes=1024
```

- 限制输出日志大小为 1024 字节。
- 可用于避免一次性输出过多日志。

------

### 11. 查看 Deployment/StatefulSet 管理的 Pod 日志

```bash
kubectl logs deployment/my-deployment
```

```bash
kubectl logs statefulset/my-statefulset
```

- 直接通过控制器名称查看其管理的任意一个 Pod 的日志（通常是第一个）。
- 也可结合 `-c`、`-f` 等参数使用。

> ⚠️ 注意：如果 Deployment 有多个 Pod，只显示其中一个的日志。如需查看所有，可用标签选择。

------

## 🛠 三、实用技巧

### ✅ 查看最近重启的 Pod 的错误日志

```bash
kubectl get pods | grep my-app
kubectl logs my-app-xxxxx --previous
```

### ✅ 实时跟踪 Deployment 的日志

```bash
kubectl logs -f deployment/my-app -n prod
```

### ✅ 查看所有匹配标签的 Pod 的日志（并行）

```bash
kubectl logs -l app=backend --all-containers=true
```

- `--all-containers=true`：如果 Pod 有多个容器，显示所有容器日志。

------

## 📚 四、常见选项总结

| 选项                         | 说明                      |
| ---------------------------- | ------------------------- |
| `-f`, `--follow`             | 实时跟踪日志              |
| `-p`, `--previous`           | 查看上一个容器实例的日志  |
| `--tail=N`                   | 仅显示最后 N 行           |
| `--since=1h`                 | 显示最近1小时内的日志     |
| `--since-time=...`           | 从指定时间开始的日志      |
| `-c`, `--container=NAME`     | 指定容器名称              |
| `-n`, `--namespace=NAME`     | 指定命名空间              |
| `-l`, `--selector=KEY=VALUE` | 根据标签选择 Pod          |
| `--max-log-requests=N`       | 最大并发获取日志的 Pod 数 |
| `--limit-bytes=N`            | 限制日志输出大小（字节）  |
| `--all-containers=true`      | 输出 Pod 中所有容器日志   |

------

## ⚠️ 五、注意事项

1. **Pod 必须存在**：如果 Pod 已被删除，无法查看日志。
2. **日志存储在节点上**：日志来自 Kubelet 管理的容器运行时（如 Docker、containerd），节点清理或日志轮转后可能丢失。
3. **生产环境建议使用日志收集系统**：
   - 如：**EFK**（Elasticsearch + Fluentd + Kibana）
   - 或 **阿里云 SLS**（日志服务）
   - 实现日志持久化、检索、告警。

------

## ✅ 六、示例：综合使用

```bash
# 查看生产环境中订单服务的实时日志（最近100行）
kubectl logs -f deployment/order-service --tail=100 -n production

# 查看所有标签为 app=api 的 Pod 的最后10行日志
kubectl logs -l app=api --tail=10 --max-log-requests=5

# 查看 sidecar 容器的日志（上一个实例）
kubectl logs my-pod -c sidecar --previous
```

------

## ✅ 总结

`kubectl logs` 是调试 Kubernetes 应用最常用的命令之一。掌握它的各种参数，能极大提升排查问题的效率。

> 🔔 **提示**：结合 `kubectl describe pod` 和 `kubectl get pods` 一起使用，能更全面地了解 Pod 状态。