### 报错信息：

```bash
错误状态：
The deployment default/gateway has 3 unavailable replicas
ExceptionContext: event, Reason: Failed, Message: Error: ImagePullBackOff
ExceptionContext: event, Reason: Failed, Message: Failed to pull image "registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway:20250805": failed to pull and unpack image "registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway:20250805": failed to resolve reference "registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway:20250805": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```



### 错误信息分析：

这是一个典型的 **镜像拉取失败错误（ImagePullBackOff）**，具体错误信息如下：

```bash
pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

说明 Kubernetes 在尝试从私有镜像仓库 `registry-vpc.cn-shenzhen.aliyuncs.com` 拉取镜像时，**鉴权失败或镜像不存在**。



### 可能原因分析：

| 原因                                   | 说明                                                         |
| :------------------------------------- | :----------------------------------------------------------- |
| **镜像仓库地址错误**                   | 镜像地址拼写错误，如仓库名、项目名、tag 错误                 |
| **镜像不存在**                         | 指定的镜像 `registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway:20250805` 实际上在仓库中不存在 |
| **未配置 imagePullSecrets**            | 私有仓库需要认证信息，但 Deployment 中未配置 `imagePullSecrets` |
| **Secret 配置错误或权限不足**          | Secret 中的认证信息错误，或对应的 RAM 用户权限不足           |
| **使用 VPC 内网地址但节点不在 VPC 内** | 使用了 `registry-vpc.cn-shenzhen.aliyuncs.com`（内网地址），但节点在公网，无法访问内网镜像仓库 |
| **容器安全策略限制（ACK 安全策略）**   | 如开启了 `ACKAllowedRepos` 或 `ACKImageDigests`，但镜像不符合白名单或 digest 要求 |



### 修复操作建议

###  1. **确认镜像是否存在**

登录阿里云容器镜像控制台：

- 地址：https://cr.console.aliyun.com/
- 检查 `ctscrm` 命名空间下是否存在 `gateway` 镜像；
- 查看是否存在 tag 为 `20250805` 的镜像；
- 如果没有，说明镜像未推送或 tag 错误。

### ✅ 2. **检查镜像仓库地址是否正确**

确认你使用的镜像地址是否正确：

| 类型             | 地址                                    |
| :--------------- | :-------------------------------------- |
| **公网地址**     | `registry.cn-shenzhen.aliyuncs.com`     |
| **VPC 内网地址** | `registry-vpc.cn-shenzhen.aliyuncs.com` |

如果你的 ACK 集群节点在公网或混合网络环境，**请使用公网地址**，否则无法访问。

### ✅ 3. **配置 imagePullSecrets**

如果使用的是私有镜像仓库，必须配置 `imagePullSecrets`。

#### 步骤：

1. **创建 Secret（用于拉取私有镜像）**

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-server=registry-vpc.cn-shenzhen.aliyuncs.com \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=you@example.com
```

1. **在 Deployment 中引用该 Secret**

```bash
spec:
  template:
    spec:
      imagePullSecrets:
        - name: my-registry-secret
      containers:
        - name: gateway
          image: registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway:20250805
```

### ✅ 4. **检查 Secret 权限是否足够**

- 如果你使用的是 RAM 用户，请确保该用户有拉取镜像的权限；
- 可前往阿里云容器镜像服务 → 命名空间 → 授权管理，添加 RAM 用户并赋予 `PullImage` 权限。

### ✅ 5. **检查是否受容器安全策略限制**

如果你开启了 ACK 的容器安全策略（如 `ACKAllowedRepos` 或 `ACKImageDigests`），需确认：

#### ✅ A. 是否镜像仓库被白名单允许？

- 检查是否配置了 `ACKAllowedRepos`；
- 确保 `registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/` 在白名单中；
- 示例配置：

```bash
parameters:
  repos:
    - registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/
```

#### ✅ B. 是否要求镜像必须使用 digest（如 `@sha256:xxx`）？

- 如果开启了 `ACKImageDigests` 策略，必须使用 digest 格式镜像；
- 当前使用的是 tag 格式（`20250805`），会被拒绝；
- 示例正确格式：

```bash
image: registry-vpc.cn-shenzhen.aliyuncs.com/ctscrm/gateway@sha256:xxxx
```