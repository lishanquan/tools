### 问题：

`docker`容器日志导致主机磁盘空间满了。

日志保存位置一般是：`/var/lib/docker/containers`对应的容器中。



### 解决：

#### 方法1：设置单个容器日志大小（治标）

通过配置`docker-compose.yml`文件中的`max-size`参数实现设置单个容器的日志大小上限：

```shell
logging:
      driver: "json-file"
      options:
        max-size: "2g"      # 日志文件大小
        max-file: "10"		# 日志的数量
```

重启容器之后，其日志文件的大小就被限制在2GB。



#### 方法2：设置全局配置（治根）

配置文件`/etc/docker/daemon.json`，添加`log-driver`和`log-opts`参数：

```shell
#	max-size=500m，意味着一个容器日志大小上限是500M，
#	max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。
{
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"}
}
```
重启`docker`守护进程

```shell
systemctl daemon-reload
systemctl restart docker
```