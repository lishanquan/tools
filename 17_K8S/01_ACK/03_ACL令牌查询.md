### 🔑 获取和管理 ACL 令牌

要解决当前的 403 权限问题，并继续安全配置，关键在于获取和使用具有足够权限的 ACL 令牌。

1. **查找引导令牌（Bootstrap Token）**
   新版本 Consul 的 ACL 引导令牌通常存储在 Kubernetes Secret 中。请尝试以下命令查找：

   ```bash
   # 查找可能的 ACL 相关 Secret
   kubectl get secrets -n consul-ns | grep -i acl
   # 或者查找包含 "bootstrap" 或 "token" 的 Secret
   kubectl get secrets -n consul-ns --field-selector type=Opaque -o name | xargs -I {} kubectl describe {} -n consul-ns | grep -i token
   ```

   如果找到相关的 Secret，可以用 `kubectl get secret <secret-name> -n consul-ns -o jsonpath='{.data.token}' | base64 --decode` 查看其内容。

   

2. **使用令牌进行查询**
   一旦找到令牌（例如，假设令牌为 `your-master-token-here`），您就可以用它来执行之前被拒绝的命令了：

   ```bash
   # 设置环境变量方便使用
   export CONSUL_HTTP_TOKEN="your-master-token-here"
   
   # 现在应该可以成功执行了
   kubectl exec -n consul-ns consul-server-0 -- sh -c 'CONSUL_HTTP_TOKEN="$CONSUL_HTTP_TOKEN" consul info | grep leader'
   ```

   也可以将令牌直接通过标准输入传递：

   ```bash
   echo "your-master-token-here" | kubectl exec -n consul-ns consul-server-0 -i -- consul info | grep leader
   ```

   