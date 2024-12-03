### 1.Dockerfile是什么？

- 概念：

  **Dockerfile是用来构建Docker镜像的构建文件，由一系列命令和参数构成的脚本**

- 构建三步骤：

  1. 编写Dockerfile文件
  2. docker build
  3. docker run



### 2.Dockerfile构建过程分析

##### 	入门知识：

	1. 每条保留字指令都必须为为**大写字母**并且后面要**跟随至少一个参数**；
 	2. 指令按照从上到下，**顺序执行**；	
 	3. `#`表示注释；
 	4. 每条指令都会创建一个新的镜像层，并对镜像进行提交；



##### 	Dockerfile执行流程分析：

1. `docker`从基础镜像运行一个容器；
2. 执行一条指令并对容器作出修改；
3. 执行类似`docker commit`的操作提交一个新的镜像层；
4. `docker`再基于刚提交的镜像运行一个新容器；
5. 执行`Dockerfile`中的下一条指令直到所有指令都执行完成；



##### 	注意：

​	在现在阶段，我们将`Dockerfile`、`Docker`镜像和`Docker`容器看作软件的三个不同阶段：

​	`Dockerfile`面向开发 ---> `Docker`镜像成为交付标准 ---> `Docker`容器则涉及部署和运维



### 3.Dockerfile保留字指令

`Dockerfile`保留字指令大致有以下：

1. ##### FROM

   基础镜像，即当前新镜像是基于哪个镜像创建的。

   ```dockerfile
   # 基于openjdk:8 创建镜像
   FROM openjdk:8
   ```

   

2. ##### MAINTANINER

   镜像维护者的姓名和邮箱地址

   ```dockerfile
   MAINTAINER lsq<123@163.com>
   ```

   

3. ##### RUN

   容器构建时需要运行的指令

   ```dockerfile
   RUN mkdir -p /conf/my.cn
   ```

   

4. ##### EXPOSE

   当前容器对外暴露的端口

   ```dockerfile
   # 暴露出Jenkins的所需端口
   EXPOSE 3000 50000
   ```

   

5. ##### WORKDIR

   指定在创建容器后，终端默认登录进来的工作目录

   ```dockerfile
   # 容器数据卷，用于数据保存和持久化工作
   WORKDIR /usr/local/mycat
   ```

   

6. ##### ENV

   用来在构建镜像过程中设置环境变量

   ```dockerfile
   # 用来在构建镜像过程中设置环境变量
   ENV MYCAT_HOME=/usr/local/mycat
   ```

   这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样，也可以在其它指令中直接使用这些环境变量。

   如：

   ```dockerfile
   RUN $MYCAT_HOME/mycat
   ```

   

7. ##### ADD 和 COPY

   ADD：

   将宿主机目录下的文件拷贝进镜像，并且ADD命令会自动处理URL和解压tar压缩包

   ```dockerfile
   ADD centos-6-docker.tar.xz / 
   ```

   

   COPY：

   类似ADD，拷贝文件和目录到镜像中。

   将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置

   ```dockerfile
   COPY src destCOPY ["src" "dest"]
   ```

   

8. ##### VOLUME

   容器数据卷，用于数据持久化和数据保存。

   ```dockerfile
   # 将mycat的配置文件的地址暴露出映射地址,启动时直接映射宿主机的文件夹VOLUME /usr/local/mycat
   ```

   

9. ##### CMD 和 ENTRYPOINT

   **CMD**：

   CMD的指令和RUN相似，也是两种格式：

   - `shell`格式：CMD <命令>
   - `exec`格式：CMD ["可执行文件", "参数1", "参数2"……]

   `Dockerfile`中可以有多个CMD指令，但只有最后一个生效，CMD会被`docker run `之后的参数替换。

   

   **ENTRYPOINT**：

   指定一个容器启动时要运行的命令。

   `ENTRYPOINT`的目的和`CMD`一样，都是在指定容器启动程序及参数。

   

   **区别**：

   在这里先简单说明一下区别，你可以将CMD理解为**覆盖**

   ```dockerfile
   CMD cat /conf/my.cnf
   CMD /bin/bash
   ```

   这两条指令都写在`Dockerfile`文件中，只会执行CMD /bin/bash ，而不会执行`CMD cat /conf/my.cnf`,因为`CMD /bin/bash`把上一条直接覆盖掉了。

   

   而`ENTRYPOINT`则不同，你可以将`ENTRYPOINT`简单理解为**追加**。

   

   主要体现在`docker run` 上，如果使用`dockerfile`文件中最后是`CMD`结尾，则在运行时不能够额外追加命令，否则会覆盖掉`Dockerfile`中的`CMD`命令。

   而`Dockerfile`文件中最后一行为`ENTRYPOINT`结尾时，你可以在`docker run `命令后追加一些命令.

   

   

10. ONBUILD

    当构建一个被继承的`Dockerfile`时运行命令，父镜像在被子继承后，父镜像的`onbuild`被触发。

