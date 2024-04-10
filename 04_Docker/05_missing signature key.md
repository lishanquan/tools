### missing signature key:

#### **问题原因**：

如果安装docker用的是yum install docker命令的话，下载下来的docker版本为旧版本，所以会有数字签名有问题。

#### 解决方案：

1. ##### 卸载Docker

   ```shell
   yum erase docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-selinux \
                     docker-engine-selinux \
                     docker-engine \
                     docker-ce
   ```

2. ##### 重新安装docker-ce

   ```shell
   yum install docker-ce -y
   ```

   如果出现新的报错：

   ```shell
   没有可用软件包 docker-ce。
   没有可用软件包 docker-ce-cli。
   没有可用软件包 containerd.io。
   ```

   **需要更新yum**

   ```shell
   yum update
   ```

   **换源**

   ```shell
   yum-config-manager --add-repo   http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

3. ##### 重装之后，需要重启Docker

   ```shell
   systemctl restart docker
   ```

4. ##### 这时候，如果拉取镜像还是出现问题，docker在启动容器的时候，报错

   ```shell
   Error response from daemon: unknown or invalid runtime name: docker-runc
   ```

   可以执行以下命令：

   ```shell
   grep -rl 'docker-runc' /var/lib/docker/containers/ | xargs sed -i 's/docker-runc/runc/g'
   ```

   然后再重启Docker。

5. 