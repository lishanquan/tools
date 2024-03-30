### docker run [option]:

#### option: 

- ##### -i    # 运行容器

- ##### -t    # 容器启动后，进入命令行模式

- ##### -v    # 目录映射

- ##### -d    # 守护进程（后台运行）

- ##### -p    # 端口映射

```bash
docker run -id --name=[container_name] -p 7070:8080 -v /usr/local/dir_name:/usr/local/dir_name [images_name]

# 命令解析
# 1、-id --name=[container] # 将新创建的"container_name"容器运行起来并且守护进程后台运行
# 2、-p 7070:8080 # 前面的"7070"是指的宿主机的端口，后面的"8080"是指的容器的端口
# 3、-v /usr/local/file_name:/usr/local/dir_name 
# 前面的"/usr/local/dir_name"文件路径是指的宿主机的文件路径，后面的"/usr/local/dir_name"文件路径是指的容器的文件路径
# 4、[images_name] # 镜像名称

# 命令作用
# 根据"images_name"镜像来创建一个名称为"container_name"容器，并将容器的"8080"端口和"/usr/local/dir_name"文件路径映射到
# 宿主机的"7070"端口和"/usr/local/dir_name"文件路径，创建容器完毕后，将该容器运行起来并且守护进程后台运行，如果docker中
# 没有该"images_name"镜像，则先拉取"images_name"镜像


# 进入"container_name"容器的命令行模式
docker exec -it [container_name] /bin/bash 

# 退出"container_name"容器命令行模式
exit 

```