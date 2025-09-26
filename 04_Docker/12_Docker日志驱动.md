### Docker全局日志驱动

------

### 🔧 方案：配置 Docker 日志驱动（推荐优先设置）

避免 Docker 自身的日志文件（`*-json.log`）无限增长。

#### 步骤 1：修改 Docker 全局配置（`/etc/docker/daemon.json`）

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

> - `max-size`: 单个日志文件最大 100MB
> - `max-file`: 最多保留 3 个历史文件（总约 300MB）

#### 步骤 2：重启 Docker 生效

```bash
sudo systemctl restart docker
```

> ⚠️ 所有新创建的容器都会应用此规则。



## 注意：

按照上述配置启动，报错：

```bash
unable to configure the Docker daemon with file /etc/docker/daemon.json: 
the following directives are specified both as config file and command line or config file: log-driver, log-opts
```

### ❌ 错误原因

> **`log-driver` 和 `log-opts` 同时在 `daemon.json` 文件和 Docker 启动命令行中定义了**，Docker 不允许重复配置。

这是典型的 **配置冲突** 问题：

- 你已经在 `/etc/docker/daemon.json` 中设置了日志驱动：

  ```json
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "500m",
      "max-file": "3"
    }
  }
  ```

- 但你的 Docker 服务（`dockerd-current`）**启动命令中也通过命令行参数指定了相同的配置**，导致冲突。

------

## 🔍 根本原因分析

从你的 `systemctl status docker.service` 输出可以看到：

```bash
ExecStart=/usr/bin/dockerd-current --add-runtime ... $OPTIONS ... $DOCKER_STORAGE_OPTIONS ... $ADD_REGISTRY ... $REGISTRIES
```

说明 Docker 使用的是 **RHEL/CentOS 风格的旧版 `dockerd-current`**（常见于 CentOS 7 或阿里云 ECS 默认安装的 Docker 版本）。

这类系统通常通过以下文件设置默认选项：

- `/etc/sysconfig/docker`
- `/etc/sysconfig/docker-network`
- `/etc/sysconfig/docker-storage`

而你很可能在这些文件中（尤其是 `/etc/sysconfig/docker`）**已经定义了日志相关参数**，例如：

```bash
# /etc/sysconfig/docker
OPTIONS='--log-driver=json-file --log-opt=max-size=50m --log-opt=max-file=3'
```

当你又在 `/etc/docker/daemon.json` 中重复定义时，Docker 就会报错：

> ```bash
> directives are specified both as config file and command line
> ```

------

## ✅ 解决方案

### ✅ 方案一（推荐）：**只保留 `daemon.json`，移除命令行中的日志配置**

#### 步骤 1：编辑 `/etc/sysconfig/docker`

```bash
vi /etc/sysconfig/docker
```

找到 `OPTIONS` 这一行，**删除 `--log-driver` 和 `--log-opt` 相关部分**：

❌ 错误写法（删除日志相关）：

```bash
OPTIONS='--log-driver=json-file --log-opt=max-size=50m --log-opt=max-file=3 --selinux-enabled'
```

✅ 正确写法：

```bash
OPTIONS='--selinux-enabled'
```

> 保留其他必要选项（如 `--selinux-enabled`），只删日志部分。