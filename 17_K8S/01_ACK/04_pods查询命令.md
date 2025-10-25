`kubectl get pods` 是查看 Kubernetes 集群中 Pod 状态的核心命令。

通过添加不同参数，可以获取更详细或特定的信息。

以下是常用参数及其意义：

------

### 🔹 常用参数一览

| 参数                       | 意义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `-n NAMESPACE`             | 指定命名空间（默认是 `default`） 例：`kubectl get pods -n kube-system` |
| `--all-namespaces` 或 `-A` | 查看所有命名空间的 Pod 例：`kubectl get pods -A`             |
| `-o wide`                  | 显示更多信息，如 IP、所在节点 例：`kubectl get pods -o wide` |
| `-o yaml`                  | 以 YAML 格式输出完整配置 例：`kubectl get pods <pod-name> -o yaml` |
| `-o json`                  | 以 JSON 格式输出（适合脚本解析）                             |
| `--show-labels`            | 显示 Pod 的标签（label）                                     |
| `-l KEY=VALUE`             | 按标签筛选 Pod 例：`kubectl get pods -l app=nginx`           |
| `--field-selector`         | 按字段筛选（如节点、状态） 例：`kubectl get pods --field-selector=status.phase=Running` |
| `--watch` 或 `-w`          | 实时监听 Pod 变化 例：`kubectl get pods -w`                  |
| `--watch-only`             | 仅监听，不显示当前状态                                       |
| `--no-headers`             | 不显示表头（常用于脚本） 例：`kubectl get pods --no-headers` |
| `-o custom-columns=`       | 自定义输出列 例： `kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP` |

------

### 🔹 实用示例

1. **查看所有命名空间中运行中的 Pod**

   ```bash
   kubectl get pods -A --field-selector=status.phase=Running
   ```

2. **查看某个命名空间下带标签的 Pod**

   ```bash
   kubectl get pods -n myapp -l env=prod --show-labels
   ```

3. **监控 Pod 状态变化**

   ```bash
   kubectl get pods -n dev -w
   ```

4. **仅输出 Pod 名称和 IP**

   ```bash
   kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP
   ```

5. **检查某个 Pod 的详细配置**

   ```bash
   kubectl get pods <pod-name> -o yaml
   ```

------

### 📌 小贴士

- 使用 `-A` 可避免遗漏系统 Pod（如 `kube-system` 中的）。
- `--no-headers` 配合 `-o wide` 常用于自动化脚本。
- `custom-columns` 非常适合定制监控或导出数据。

掌握这些参数，能让你更高效地排查问题和管理集群。