```bash
# 查看本地镜像
docker images 

# 查看"images_name"镜像源中镜像所有可下载的版本
docker search [images_name] 

# 拉取"images_name"镜像源中最新的版本
docker pull [images_name] 

# 拉取"images_name"镜像源中指定的"version_number"版本
docker pull [images_name]:[version_number] 

# 删除"images_name"镜像
docker rmi [images_name] 

# 根据"images_name"创建一个名称为"container_name"的容器
docker create --name=[container_name] [images_name] 

# 查看正在运行的容器
docker ps 

# 查看所有容器
docker ps -a

# 启动"container_name"容器
docker start [container_name] 

# 停止"container_name"容器
docker stop [container_name] 

# 删除"container_name"容器（无法删除正在运行的容器）
docker rm [container_name] 

# 强制删除"container_name"容器（可以删除正在运行的容器）
docker rm -f [container_name] 

# 删除所有容器
docker rm $(docker ps -a -q) 

# 停止所有容器
docker stop $(docker ps -a -q) 

```